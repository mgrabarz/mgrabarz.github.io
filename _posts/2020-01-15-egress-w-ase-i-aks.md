---
title: Przemyślenia na temat ruchy wychodzącego w ASE i AKS
date: 2020-01-15T22:00:00+01:00
header:
  teaser: /assets/images/2020/01/clock-time-time-of-time-indicating-pointer-watches.jpg
categories:
  - Security
tags:
  - Compute
  - ASE
  - Firewall
  - NTP
  - Storage
---

Praca dla dużych przedsiębiorstw używających chmury wymaga sporej ilości pracy związanej z zapewnieniem wysokich standardów bezpieczeństwa usług. Zwykle w firmach zdefiniowane są restrykcyjne polityki, powiązane z typem czy klasyfikacją danych. Mamy więc do czynienia z danymi osobowymi, medycznymi, finansowymi, czy chociażby takimi, które po ujawnieniu pozbawią firmę przewagi konkurencyjnej. Nie oznacza to bynajmniej, że nie powinniśmy dbać o bezpieczeństwo w mniejszych organizacjach. Te zwykle nie mają wystarczających zasobów, czasu, lub też błędnie uważają, że ryzyko ich nie dotyczy.
{: style="text-align: justify;"}

Jakiś czas temu u jednego z moich klientów konfigurowaliśmy ASE (App Service Environment) i AKS (Azure Kubernetes Service) w sieci prywatnej. Kluczowe jest nie tylko zapewnienie poprawnego filtrowania ruchu na wejściu do klastra, dalej pomiędzy usługami, ale też na ruchu wychodzącym. W takiej sytuacji punktem wyjścia mogą być następujące artykuły:
{: style="text-align: justify;"}
- Dla ASE [https://docs.microsoft.com/en-us/azure/app-service/environment/firewall-integration(https://docs.microsoft.com/en-us/azure/app-service/environment/firewall-integration)
- Dla AKS [https://docs.microsoft.com/en-us/azure/aks/limit-egress-traffic](https://docs.microsoft.com/en-us/azure/aks/limit-egress-traffic)

### Dlaczego warto filtrować ruch wychodzący?

Standardowa konfiguracja ASE i AKS nie filtruje ruchu wychodzącego. Dużo się mówi o wystawianiu portów, load balancerów oraz publikacji API na świat. Równie popularna jest tematyka mikrosegmentacji sieci i usług. O ruchu wychodzącym (egress) informacji jest znacznie mniej. 
{: style="text-align: justify;"}
Załóżmy jednak, że z powodu niedopatrzenia, podatności czy celowego działania kod naszej usługi (hostowanej na ASE lub AKS) mając dostęp do danych wrażliwych zaczyna je transferować poza naszą infrastrukturę. Ostatecznie przecież w dużej ilości ataków na firmy chodzi o to, by coś z nich ukraść, a nie popsuć dla zabawy. W przeciwnym razie taki atak byłby po prostu nieopłacalny. Mając otwarte porty narażamy się właśnie na wycieki czy też tak zwaną eksfiltrację danych.
{: style="text-align: justify;"}

### Problemy z NTP w ASE

Przeanalizujmy dwa przytoczone artykuły. Na pierwszy rzut oka wszystko wygląda jak należy (przyznam, że było kilka braków na liście, które musieliśmy dodać poprzez support i grupy produktowe). W artykule o ASE niestety rzuca się w oczy następująca tabela:
{: style="text-align: justify;"}

| Endpoint | Details |
|---|---|
| *:123 | NTP clock check. Traffic is checked at multiple endpoints on port 123 |
| *:12000 | This port is used for some system monitoring. If blocked, then some issues will be harder to triage but your ASE will continue to operate |

Tak jak port 12000 można zablokować kosztem bliżej nieokreślonych funkcji monitorowania klastra, tak port 123 musi być otwarty. To z kolei jest w pełni nieakceptowalne. Nasze dane mogą wyciekać właśnie po tym porcie do infrastruktury atakującego, bez najmniejszej weryfikacji docelowych adresów IP i FQDN.
{: style="text-align: justify;"}

Zaryzykowaliśmy blokadę i po paru tygodniach mieliśmy rozsynchronizowany czas na węzłach klastra, w niektórych przypadkach sięgający kilku minut. To z kolei wróciło w postaci bugów, jak chociażby:
{: style="text-align: justify;"}

- "Przedłużony" czas życia tokenów JWT
- Przedwczesne wygasanie SAS (Shared Access Signature) do Azure Storage
- Brak synchronizacji czasu w logach i telemetrii

Z jakiegoś powodu ASE nie używa **VMICTimeSync**, usługi odpowiedzialnej za synchronizację czasu pomiędzy maszynami wirtualnymi i ich hostem. Jeżeli nawet wykorzystuje tryb mieszany (domyślny), to nie tłumaczy to tak dużych różnic pomimo braku dostępu do zewnętrznych usług NTP. Więcej na ten temat znajdziesz [tutaj](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/time-sync) i [tutaj dla Linuxów](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/time-sync).
{: style="text-align: justify;"}

Częściowym rozwiązaniem problemu jest podkręcenie reguł opisanych w dokumentacji dla zabezpieczenia ASE i zamiast wypuszczania całego ruchu na porcie 123, wystarczy wypuścić ruch do serwerów NTP Windows i Ubuntu (jeśli używasz Linuxowych workerów w ASE):
{: style="text-align: justify;"}

```text
ntp.ubuntu.com
    - 91.189.89.199
    - 91.189.89.198
    - 91.189.94.4
    - 91.189.91.157
# Pool Twojego regionu, w którym działa VMka: https://www.ntppool.org/zone/@
    - IP1
    - IP2
    - IP3
    - IP4
# time.windows.com
    - Bez statycznych adresów...
```

### Problemy w AKS

Zdecydowaną gwiazdą w opisie blokowania ruchu wychodzącego jest reguła związana z ACR (Azure Container Registry). Pozostaje się tylko zastanowić czy nie dało się tego lepiej zaprojektować :)
{: style="text-align: justify;"}

| FQDN | Port | Use |
|---|---|---|
| *.blob.core.windows.net | HTTPS:443 | This address is the backend store for images stored in ACR. |

Obecnie nie mamy innej możliwości niż wypuścić ruch do dowolnego Storage Account w Azure! Aż trudno w to uwierzyć. Microsoft wie o tym ograniczniu ale jak widać nie zostało to dotychczas zmienione. Tutaj przykład wesołych kometarzy...
{: style="text-align: justify;"}

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">AKS egress lockdown is now generally available. 🔒<a href="https://t.co/i9sf9uQtnJ">https://t.co/i9sf9uQtnJ</a></p>&mdash; Gabe Monroy (@gabrtv) <a href="https://twitter.com/gabrtv/status/1174396584663994369?ref_src=twsrc%5Etfw">September 18, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

### Podsumowanie

Kontrola ruchu wychodzącego w obecnych czasach jest niezbędna. Dane mogą wyciekać na nietypowych portach jak NTP, czy nawet przez DNS. Warto w tym miejscu stosować metody, które przez wielu mogą być uznane za paranoję. Blokujemy wszystko i stopniowo budujemy swoją whitelistę.
{: style="text-align: justify;"}

Dobrą metodą jest cykliczne przeglądanie logów zapory z celu odnalezienia powtarzających się wzorców. Z czasem, poza oficjalną dokumentacją, znajdziemy adresy do których próbują sie dostać nasze usługi. Po analizie wywstarczy je odblokować i dodać na listę dozwolonych adresów czy FQDNów.
{: style="text-align: justify;"}
