---
title: Standardy nazewnicze zasobów w Azure
date: 2017-06-14T19:15:40+02:00
header:
  teaser: /assets/images/2017/06/checklist-2077021_640.jpg
categories:
  - Governance
tags:
  - Azure
  - Governance
  - Naming
  - Standards
  - Subscription
---
Jeżeli dopiero zaczynasz swoją przygodę z Azure, pewnie po utworzeniu kilku/kilkunastu zasobów zorientujesz się, że potrzebujesz wdrożyć standardy nazewnicze. Przy większej ilości subskrypcji i zasobów tworzonych bez żadnych konwencji zrobienie czegokolwiek w Azure robi się dość skomplikowane. Poza ogarnięciem wszystkiego w pamięci (a pewnie nie tylko Ty będziesz tworzył komponenty i je używał) nie masz zbyt wielu możliwości, a pomylić możesz się dosłownie na każdym kroku.
{: style="text-align: justify;"}

W dzisiejszym wpisie przybliżę stosowaną przeze mnie konwencję nazewniczą zasobów w Azure oraz postaram się w skrócie zahaczyć o konwencję nazewniczą dla subskrypcji w EA portalu. Poniżej opisane podejście bazuje dość mocno na <a href="https://docs.microsoft.com/en-us/azure/architecture/best-practices/naming-conventions" target="_blank" rel="noopener">wytycznych</a> Microsoft i zostało przeze mnie wdrożone w kilku organizacjach. Zapewniam Cię, że efekt jaki osiągniesz, w szczególności uproszczenie pracy w Azure, jest niesamowity!
{: style="text-align: justify;"}

### Dlaczego standardy nazewnicze są istotne?

W zasadzie odpowiedź na to pytanie nasuwa się sama. Jak w przypadku większości standardów pozwalają one uniknąć dwuznaczności i ułatwiają one pracę. Oto kilka dodatkowych powodów dla których warto je wdrożyć:
{: style="text-align: justify;"}

- Wszystkie osoby pracujące z Azure w Twojej organizacji, zaczynając od deweloperów i kończąc na liniach wsparcia, będą posługiwać się wspólnym językiem na temat zasobów. Nie jest ważne kto i kiedy stworzył zasób, jego nazwa natychmiast daje odpowiedź dotyczącą powodu jego istnienia.
- We wszystkich logach lądują czytelne nazwy. Bez tego dowiesz się, że "website3344" się wyłożył(a), pewnie z powodu "appdatabejz-56", albo "MojRedis", najpewniej jednak winowajcą jest "myresource".
- Prawdopodobieństwo zrobienia błędu w portalu, skrypcie albo szablonie ARM jest zdecydowanie mniejsze.
- Łatwo odróżnisz produkcję od środowiska testowego, albo lokalizację zasobu.
- Łatwo odróżnisz typ zasobów po nazwie, a nie po ikonce w Portalu.
- Zyskasz mnóstwo czasu, który do tej pory był tracony na wyszukiwanie zasobów. Przy użyciu standardów nazewniczych w większości przypadków nazwę zasobu można po prostu zgadnąć.

### Nazwy subskrypcji w EA

Duże organizacje używające Azure dość często decydują na podpisanie z Microsoft tzw. Enterprise Agreement, co skutkuje zarządzaniem subskrypcjami z poziomu portalu EA (ea.azure.com). Sposób budowy struktury wielu subskrypcji jest mocno uzależniony od specyfiki organizacji. Trzy standardowe podejścia są dość dobrze opisane w <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-subscription-governance#define-your-hierarchy" target="_blank" rel="noopener">Azure enterprise scaffold &#8211; prescriptive subscription governance</a>.
{: style="text-align: justify;"}

Nie będę wchodził w szczegóły struktury EA u mojego obecnego pracodawcy, same jednak nazwy subskrypcji mają aktualnie następujący format: 
{: style="text-align: justify;"}

```html
<ProjectName>_<OrgName>_<Stage>
```

**ProjectName**: Nazwa projektu (w <a href="https://pl.wikipedia.org/wiki/PascalCase" target="_blank" rel="noopener">PascalCase</a>), czy też większego portfela projektów (np subskrypcja integracyjna, czy korporacyjny API Management).
{: style="text-align: justify;"}

**OrgName**: Nazwa firmy córki w grupie kapitałowej, lub departamentu w przypadku największej z firm córek. Taki akurat podział wynika ze struktury firmy, w której pracuję. W większości przypadków nazwa działu/departamentu będzie wystarczająca.
{: style="text-align: justify;"}

**Stage**: Tutaj dopuszczamy dwie opcje Prod albo NonProd. Do subskrypcji produkcyjnych wpuszczamy osoby upoważnione, w tych nieprodukcyjnych oddajemy "sterowanie" liderom projektów i osobom spoza ścisłego, zaufanego grona. W NonProdach używamy innego i tańszego typu subskrypcji (a konkretnie Dev/Test), możemy wdrożyć ARM Policies na typ/lokalizację/tier zasobów. Bez obaw możemy dodać dewelopera jako co-admina w celu umożliwienia dostępu do starego portalu (na szczęście coraz rzadziej) - bez podziału na Prod/NonProd uzyskałby on dostęp do produkcji.
{: style="text-align: justify;"}

### Standardy nazewnicze zasobów w subskrypcji

Używany przeze mnie format w przypadku grup zasobów i samych zasobów jest nieco zbliżony do powyższego, dotyczącego samych subskrypcji:
{: style="text-align: justify;"}

```html
<ProjectName>-<Stage>-<Location>-<ResourceType>-<Description>
```

**ProjectName**: Nazwa projektu, do którego należy dany zasób. Często pokrywa się z nazwą projektu w nazwie subskrypcji.
{: style="text-align: justify;"}

**Stage**: Zdefiniowany wcześniej słownik skrótów, które reprezentują typ środowiska. Może się tu pojawić Prod, Test, Dev, Uat, Hotfix itp.
{: style="text-align: justify;"}

**Location**: Trzyliterowy skrót słownikowy (czasem z dodatkową cyfrą) opisujący region, w którym znajduje się zasób. Dla Europe West "euw", US East 2 "use2" itp.
{: style="text-align: justify;"}

**ResourceType**: W tym miejscu również proponuję słownik, który reprezentuje typ zasobu. Dla grupy zasobów może to być "rg", dla sieci wirtualnej "vnet" dla AppService Plan "plan" itp.
{: style="text-align: justify;"}

**Description**: Skrót opisujący cel istnienia zasobu, umożliwia on odróżnienie zasobów tego samego typu w jednej subskrypcji.
{: style="text-align: justify;"}

Dla przykładu, ten blog mógłby składać się z następujących zasobów:

- personalweb-prod-eun-webapp-wphosting:  **WebApp**, umiejscowiony w **North Europe**, odpowiedzialny za **produkcyjny** hosting **WordPressa** , będący częścią projektu **personalweb**.
- personalweb-dev-eun-appins-wphoting: **Application Insights**, umiejscowione w **North Europe**, zbierający dane z **deweloperskiego WordPressa**, będący częścią projektu **personalweb**.

### Podsumowanie i drobne uwagi

Każda reguła ma często swoje odstępstwa, pojawią się one również tutaj.

- Pierwszym problemem jaki możesz napotkać jest brak możliwości określenia jednego z wymaganych sufiksów. Przykładem może być Traffic Manager, który w polu Location będzie miał "Global" - inne tego typu problemy pozostawiam Tobie, da się je spokojnie rozwiązać.
{: style="text-align: justify;"}
- Drugim, poważniejszym wyzwaniem są <a href="https://docs.microsoft.com/en-us/azure/architecture/best-practices/naming-conventions#naming-rules-and-restrictions" target="_blank" rel="noopener">ograniczenia</a> dla nazw poszczególnych typów zasobów w samym Azure. Największą zakałą jest tu Storage Account (przez to również Data Lake), który nie dopuszcza myślników w nazwie. Pomimo, że jego nazwa wewnętrznie jest zgodna z RFC 1123, grupa produktowa celowo zablokowała możliwość używania tego znaku "zwykłym" użytkownikom. Ze względu na swoją wygodę trochę utrudnili nam życie i nie planują tego zmienić w przewidywalnej przyszłości! Na pocieszenie mamy "-secondary" z myślnikiem w opcji RA-GRS 🙂
{: style="text-align: justify;"}
- Byłoby świetnie, gdyby tego typu konwencję dało się wdrożyć przy pomocy Azure Resource Policies. Niestety są one w chwili obecnej zbyt ograniczone, by dało się te reguły przy ich pomocy wyrazić. Kolejna moja dyskusja z grupą produktową zaowocowała propozycją, by dodali do zestawu <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-policy#policy-rule" target="_blank" rel="noopener">dostępnych Conditions</a> opcję RegEx. Czekam aż mój pomysł kiedyś dotrze na szczyt ich backlogu. Może skorzystać z UserVoice?!
{: style="text-align: justify;"}

Mam nadzieję, że przedstawione powyżej standardy nazewnicze pomogą Ci w przyszłości skutecznie zapanować nad tworzonymi zasobami w Azure. **Jeżeli masz już wdrożony standard, napisz proszę w komentarzach jak on wygląda!** Może uda się nam połączyć wszystko co najlepsze i stworzyć coś naprawdę unikalnego i wygodnego w użyciu.
{: style="text-align: justify;"}