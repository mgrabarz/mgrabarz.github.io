---
title: ARMClient - Świetne narzędzie do zarządzania zasobami w Azure
date: 2017-07-18T07:33:04+02:00
header:
  teaser: /assets/images/2017/07/tools-spanner-mechanic.png
categories:
  - Tools
tags:
  - ARM
  - ARMClient
  - Azure
  - SSL
  - tools
---
Inspiracją do dzisiejszego wpisu jest <a href="http://architektwchmurze.pl/narzedzia/" target="_blank" rel="noopener">zestawienie </a>przydatnych narzędzi w Azure stworzone przez Michała Furmankiewicza na jego blogu. Lista zwiera kilka perełek i zdecydowanie warto się jej przyjrzeć, na pewno znajdziecie coś użytecznego dla siebie.
{: style="text-align: justify;"}

Moim kandydatem do listy jest ARMClient, małe konsolowe narzędzie <a href="https://github.com/projectkudu/ARMClient" target="_blank" rel="noopener">rozwijane </a>w modelu OSS. Zasada jego działania jest w sumie dość prosta. Po zalogowaniu wysyła ono zlecone żądania do RESTOwege API Azure, a konkretnie do Azure Resource Manager API.  W zasadzie narzędzie nie robi niczego więcej, poza tym że upraszcza dostęp do kluczowego chmurowego API, którym pod spodem posługują się między innymi szablony ARMowe, Portal Azure, xplatcli albo Azure Powershell.  Więcej na ten temat znajdziesz pod <a href="https://docs.microsoft.com/pl-pl/azure/azure-resource-manager/resource-group-overview" target="_blank" rel="noopener">tym </a>adresem.
{: style="text-align: justify;"}

## W jakich sytuacjach przydaje się ARMClient?

Odpowiedź na to pytanie będę starał się pokazać w oparciu o kilka przykładów. W zasadzie zakres użycia narzędzia jest bardzo szeroki, tak jak samo ARM API. To wszystko, co zwykle robisz przy użyciu wymienionych powyżej narzędzi (np. xplatcli), będziesz mógł również wykonać przy użyciu ARMClient. Często zdarza się też, że nowe, ukryte funkcje poszczególnych typów zasobów nie są jeszcze widoczne na Portalu Azure, albo nie są jeszcze zaimplementowane w poleceniach Azure Powershell, natomiast z punktu widzenia ARM API są i czekają na użycie. Podobnie, gdy wyklikamy coś w Portalu bez późniejszej możliwości zautomatyzowania kroków, uświadamiamy sobie, że te zmiany są dokonywane przez API ARMowe, czyli możemy klikanie zastąpić prostymi wywołaniami z narzędzia.
{: style="text-align: justify;"}

Żeby zobrazować Ci zakres możliwości proponuję zajrzeć na dość dobrze znany <a href="https://resources.azure.com" target="_blank" rel="noopener">resources.azure.com</a>. Na stronie przedstawiona jest struktura Twoich zasobów w Azure w postaci zagnieżdżonych elementów reprezentowanych przez elementy w JSON. Warto wspomnieć, że strukturę tę możesz przeglądać pod różnymi kątami:
{: style="text-align: justify;"}

- W hierarchii subskrypcja/grupa_zasobów/provider/zasób
- Subskrypcja/provider/zasób
- Provider/zasób

Czym są providery? Jeśli przeczytałeś artykuł podany w ostatnim linku w poprzednim paragrafie, to pewnie już wiesz. W uproszeniu są to pluginy do API ARM, które odpowiedzialne są za obsługę poszczególnych typów zasobów, np:
{: style="text-align: justify;"}

- **Microsoft.Web** odpowiada za AppService i tematy pochodne (certy, domeny, bindingi, config, connectionstrings itp)
- **Microsoft.Compute** za maszyny wirtualne, w tym również dyski, scalesety.
- **Microsoft.Storage** za konta magazynów.

No dobrze, ale jak się to ma do naszego narzędzia? Otóż za każdym razem jak oglądamy dokumenty JSON, na stronie pojawia się zielony guzik GET wraz z adresem URL. Przy pomocy ARMClienta możemy zrobić dokładnie to samo, jedynie kopiujemy link do konsoli. Poniżej pobranie listy subskrypcji:
{: style="text-align: justify;"}

```powershell
armclient GET /subscriptions?api-version=2014-04-01
```

Sztuczka polega na tym, że ARMClient obsługuje również pozostałe polecenia HTTP, konkretnie POST, PUT, PATCH i DELETE. To z kolei sprawia, że możemy czytać poszczególne zasoby, wraz z najdrobniejszymi ustawieniami konfiguracyjnymi, zmieniać te ustawienia i z powrotem zapisywać w chmurze.
{: style="text-align: justify;"}

## Jak zainstalować ARMClient?

W związku z tym, że projekt rozwijany jest w klasycznym .NET, raczej nie ma szans odpalenia go poza Windowsami.
{: style="text-align: justify;"}

Twórcy aplikacji zalecają, by instalacji dokonywać przy pomocy <a href="https://chocolatey.org/" target="_blank" rel="noopener">Chocolatey</a>. Jeżeli jeszcze nie znasz **choco**, to najwyższa pora byś je u siebie dodał. Jest to świetny system zarządzania pakietami, który swoimi możliwościami zbliżony jest do znanych z Linuxa apt-get czy yum. Odpalamy konsolę z uprawnieniami admina i wykonujemy następujące polecenie
{: style="text-align: justify;"}

```powershell
choco install ARMClient
```

Po zakończonej instalacji możemy zacząć wysyłać polecenia do ARM API, zanim to jednak będzie możliwe musimy się zalogować. Narzędzie obsługuje tryb interaktywny (z okienkiem logowania):
{: style="text-align: justify;"}

```powershell
ARMClient login
```

Jak również tryb cichy, używany w przypadku automatyzacji zadań i wykonywania cyklicznych skryptów przy użyciu kont użytkowników/technicznych autoryzowanych w Azure i w naszych subskrypcjach:
{: style="text-align: justify;"}

```powershell
#Tryb appId i Key, dla aplikacji AAD reprezentowanej przez Service Principal Name
ARMClient spn [tenant] [appId] (appKey) 

#Również SPN, tym razem z certyfikatem zamiast klucza
ARMClient spn [tenant] [appId] [pfxFile] (cert_password) 

#Ciche logowanie przez UPN. UPN musi należeć do Tenanta AAD.
ARMClient upn [username] (password)
```

## Przykładowe zastosowania ARMClient

W związku z tym, że operujemy na ARM API, możliwości jest naprawdę wiele i często rodzą się one w odpowiedzi na jakieś tematy, których nie da się oskryptować.
{: style="text-align: justify;"}

**Przyklad #1**

Ostatnio na naszej facebookowej grupie i równolegle u mnie w pracy pojawiło się pytanie, jak stworzyć listę wszystkich certyfikatów SSL rozrzuconych w subskrypcjach Azure. Idealnie byłoby również, by skrypt wyświetlał wszystkie przeterminowane certy oraz te, które wkrótce wymagają odnowienia. Nie jestem nawet pewien, czy coś takiego da się zrobić w Azure Powershell, wiem natomiast, że provider Microsoft.Web/Certificates jest w stanie takie dane wydobyć.
{: style="text-align: justify;"}

```powershell
armclient login

$subscriptions = (armclient GET /subscriptions?api-version=2014-04-01) | ConvertFrom-Json
$res=foreach ($subscription in $subscriptions.value)
{
    $armWebCertificateRequestPath = '/subscriptions/' + $subscription.SubscriptionId + '/providers/Microsoft.Web/certificates?api-version=2016-03-01'

    $certificates = (armclient GET $armWebCertificateRequestPath) | ConvertFrom-Json    
    $certificates | Where-Object { [System.DateTime]::Parse($_.properties.expirationDate) -lt (Get-Date).AddDays(14)}  | select 
} 
$res| Out-GridView
```

Pierwsze uruchomienie ARMClient pobiera listę subskrypcji, następnie iteracyjnie w każdej z subskrypcji pobiera listę certyfikatów. Zagnieżdżona pętla sprawdza datę wygaśnięcia certyfikatu (14 dni do przodu) i dodaje do listy wygasających. Na uwagę zasługuje polecenie __ConvertFrom-Json__, które w przypadku ARMClient w Powershell będzie nadużywane (podobnie jak odpowiednik __ConvertTo-Json__. Innym ciekawym poleceniem jest __Out-GridView__, który wyświetla wyniki w tabelce, dzięki <a href="https://twitter.com/jnowwwak" target="_blank" rel="noopener">Janusz </a>za rekomendację.
{: style="text-align: justify;"}

**Przykład #2**

Od jakiegoś czasu możemy wymusić na kontach magazynu, żeby zawsze używały HTTPS. Niestety taka możliwość pojawiła się w Powershell (Set-AzureRMStorageAccount) dopiero w najnowszej wersji AzureRm.Storage v3.0. Jak możemy zmienić to ustawienie dla wszystkich naszych kont w ramach subskrypcji? Nic prostszego, wystarczy użyć ARMClient:
{: style="text-align: justify;"}

```powershell
armclient login

$storageAccounts = (armclient GET subscriptions/id_twojej_subskrypcji/providers/Microsoft.Storage/storageAccounts?api-version=2017-06-01) | ConvertFrom-Json
foreach ($account in $storageAccounts.value)
{
    $account.properties | Add-Member -Force "supportsHttpsTrafficOnly" True
    $resourcePath = $account.id + "?api-version=2017-06-01"
    $account | ConvertTo-Json | armclient PUT $resourcePath
}
```

W tym przypadku używamy komendy PUT w celu zapisania zmian do ARM. Poza tym wspomniane wcześniej serializator i deserializator JSON.
{: style="text-align: justify;"}

## Podsumowanie

Jak już pewnie zauważyłeś, przedstawione narzędzie jest bardzo proste w obsłudze. Każde kolejne wywołanie wygląda praktycznie identycznie, jedyna zmienna to adres żądania. Adres oczywiście łatwo jest samemu złożyć, lub zwyczajnie skopiować z resources.azure.com.
{: style="text-align: justify;"}

W związku z tym, że wchodzimy w niskopoziomowe API Azure, warto z rozwagą posługiwać się narzędziem. Dobrze przetestuj swoje skrypty na środowisku deweloperskim zanim uruchomisz je na produkcji.
{: style="text-align: justify;"}

Mam nadzieję, że ARMClient będzie bardzo przydatny w Twojej pracy i skróci czas potrzebny na implementację niektórych skryptów. Powodzenia!
{: style="text-align: justify;"}

Photo via [Visualhunt.com](https://visualhunt.com/re/8cad28)
{: style="text-align: justify;"}