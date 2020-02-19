---
title: Zmiany w regułach filtrowania dla AKS i MCR
date: 2020-02-18T01:30:00+01:00
header:
  teaser: /assets/images/2020/02/firewall.jpg
categories:
  - Security
tags:
  - Compute
  - Firewall
  - AKS
  - ACR
  - Egress
  - MCR
---

[Miesiąc temu](https://grabarz.pl/security/egress-w-ase-i-aks/) narzekałem na potrzebę dodawania **.blob.core.windows.net** do reguł ruchu wychodzącego w AKS. Okazuje się, że przed kilkoma dniami doczekaliśmy się oficjalnie poprawek.
{: style="text-align: justify;"}

Według obecnej wersji dokumentacji wystarczy teraz postawić nowego AKSa lub wykonać aktualizację klastra (np przy pomocy **az aks upgrade**) i zapomnieć o wystawianiu się na cały storage, a przy okazji na **aksrepos.azurecr.io**. Oba wpisy możemy spokojnie usunąć z reguł zapory, ale zanim to zrobimy musimy wykonać kilka kroków sprawdzających.
{: style="text-align: justify;"}

### Gdzie się podziały reguły dla MCR?

Nasuwa się w tym kontekście dość oczywiste pytanie. Usuwamy wpis dla [Microsoft Container Registry](https://github.com/microsoft/containerregistry) ale nic nie dodajemy na jego miejsce. Jakim cudem AKS ma teraz ściągać obrazy?
{: style="text-align: justify;"}
Takie wpisy powinny JUŻ znajdywać się w Twoich regułach. Są to odpowiednio:

| FQDN | Port | Use |
|---|---|---|
| mcr.microsoft.com | HTTPS:443 | This address is required to access images in Microsoft Container Registry (MCR). This registry contains first-party images/charts(for example, moby, etc.) required for the functioning of the cluster during upgrade and scale of the cluster. |
| *.cdn.mscr.io | HTTPS:443 | This address is required for MCR storage backed by the Azure content delivery network (CDN). |

Jeżeli brakuje Ci tych wpisów, koniecznie je dodaj przed usunięciem poprzednich i aktualizacją AKSa. Ważnym wnioskiem dla Ciebie powinno być wyrobienie sobie nawyku monitorowania dokumentacji i "nadganiania" z regułami zapory zanim wykonamy aktualizację, a ta zmienia się ostatnio dość często.
{: style="text-align: justify;"}

### No dobrze ale dlaczego dwa wpisy?

Według specyfikacji [OCI](https://www.opencontainers.org) (Open Container Initiative) distribution na operację  **docker image pull** składa się:
{: style="text-align: justify;"}

* Ściągnięcie manifestu obrazu. Do tego potrzebujemy RESTowego GETa na mcr.microsoft.com. Adres ten jest balansowany pomiędzy regionami Azure więc może przykrywać kilka instancji API.
{: style="text-align: justify;"}
* W zależności od potrzeb i stanu lokalnego cache operacja wykona też kilka GET w celu ściągnięcia brakujących layerów, z których składa się obraz. Do tego potrzebujemy CDNa i adresu *.cdn.mscr.io
{: style="text-align: justify;"}

Po więcej szczegółów odsyłam Cię do źródła: <https://github.com/opencontainers/distribution-spec/blob/master/spec.md#pulling-an-image>
{: style="text-align: justify;"}

### To nie koniec zmian - przygotuj się na kolejne!

3 marca 2020 *.cdn.mscr.io zostanie zastąpiony przez *.data.mcr.microsoft.com. Zmiana ta ma na celu ujednolicenie adresów używanych przez MCR. Dodatkowo nowy FQDN będzie trzymał się konwencji dla swojego CDNa, to jest [region].data.mcr.microsoft.com.
{: style="text-align: justify;"}

Poniższa tabela przedstawia ostateczną listę potrzebnych wpisów:
{: style="text-align: justify;"}

| Protokół | Adres FQDN | Uwagi |
|---|---|---|
| https | mcr.microsoft.com | Wymagane dzisiaj |
| https | *.cdn.mscr.io | Przestanie być używane od 3 marca 2020 |
| https | *.data.mcr.microsoft.com | Zacznie być używane 3 marca 2020, ale warto dodać już dzisiaj! |

### Podsumowanie

* Wykonaj aktualizację reguł na zaporze w najbliższym czasie. Dzięki temu krokowi odcinamy wyjście do wszystkich kont magazynu Azure, a jednocześnie szykujemy się na zmiany 3 marca.
{: style="text-align: justify;"}
* Powyższe wymagania nie dotyczą jedynie AKS. Te same zmiany wpłyną na funkcjonowanie ASE i innych usług kontenerowych w Azure działających za zaporą.
{: style="text-align: justify;"}
* Bądź na bieżąco ze zmianami w dokumentacji i dokonuj odpowiednich korekt w swojej konfiguracji.
{: style="text-align: justify;"}

Photo credit: <a href="https://visualhunt.co/a4/09e67b">Yu. Samoilov</a> on <a href="https://visualhunt.com/re6/084cb9d9">Visual Hunt</a> / <a href="http://creativecommons.org/licenses/by/2.0/"> CC BY</a>
