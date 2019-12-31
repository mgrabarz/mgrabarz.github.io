---
title: Standardy nazewnicze zasobów w Azure
date: 2017-06-14T19:15:40+02:00
image: /assets/images/2017/06/checklist-2077021_640.jpg
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

W dzisiejszym wpisie przybliżę stosowaną przeze mnie konwencję nazewniczą zasobów w Azure oraz postaram się w skrócie zahaczyć o konwencję nazewniczą dla subskrypcji w EA portalu. Poniżej opisane podejście bazuje dość mocno na <a href="https://docs.microsoft.com/en-us/azure/architecture/best-practices/naming-conventions" target="_blank" rel="noopener">wytycznych</a> Microsoft i zostało przeze mnie wdrożone w kilku organizacjach. Zapewniam Cię, że efekt jaki osiągniesz, w szczególności uproszczenie pracy w Azure, jest niesamowity!

### Dlaczego standardy nazewnicze są istotne?

W zasadzie odpowiedź na to pytanie nasuwa się sama. Jak w przypadku większości standardów pozwalają one uniknąć dwuznaczności i ułatwiają one pracę. Oto kilka dodatkowych powodów dla których warto je wdrożyć:

  * Wszystkie osoby pracujące z Azure w Twojej organizacji, zaczynając od deweloperów i kończąc na liniach wsparcia, będą posługiwać się wspólnym językiem na temat zasobów. Nie jest ważne kto i kiedy stworzył zasób, jego nazwa natychmiast daje odpowiedź dotyczącą powodu jego istnienia.
  * We wszystkich logach lądują czytelne nazwy. Bez tego dowiesz się, że &#8216;website3344&#8217; się wyłożył(a), pewnie z powodu &#8220;appdatabejz-56&#8221;, albo &#8220;23_MojRedis&#8221;, najpewniej jednak winowajcą jest &#8220;myresource&#8221;.
  * Prawdopodobieństwo zrobienia błędu w portalu, skrypcie albo szablonie ARM jest zdecydowanie mniejsze.
  * Łatwo odróżnisz produkcję od środowiska testowego, albo lokalizację zasobu.
  * Łatwo odróżnisz typ zasobów po nazwie, a nie po ikonce w Portalu.
  * Zyskasz mnóstwo czasu, który do tej pory był tracony na wyszukiwanie zasobów. Przy użyciu standardów nazewniczych w większości przypadków nazwę zasobu można po prostu zgadnąć.

### Nazwy subskrypcji w EA

Duże organizacje używające Azure dość często decydują na podpisanie z Microsoft tzw. Enterprise Agreement, co skutkuje zarządzaniem subskrypcjami z poziomu portalu EA (ea.azure.com). Sposób budowy struktury wielu subskrypcji jest mocno uzależniony od specyfiki organizacji. Trzy standardowe podejścia są dość dobrze opisane w <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-subscription-governance#define-your-hierarchy" target="_blank" rel="noopener">Azure enterprise scaffold &#8211; prescriptive subscription governance</a>.

Nie będę wchodził w szczegóły struktury EA u mojego obecnego pracodawcy, same jednak nazwy subskrypcji mają aktualnie następujący format: <code class="EnlighterJSRAW" data-enlighter-language="null">&lt;ProjectName&gt;_&lt;OrgName&gt;_&lt;Stage&gt;</code>

<span style="text-decoration: underline;"><ProjectName></span>: Nazwa projektu (w <a href="https://pl.wikipedia.org/wiki/PascalCase" target="_blank" rel="noopener">PascalCase</a>), czy też większego portfela projektów (np subskrypcja integracyjna, czy korporacyjny API Management).

<span style="text-decoration: underline;"><OrgName></span>: Nazwa firmy córki w grupie kapitałowej, lub departamentu w przypadku największej z firm córek. Taki akurat podział wynika ze struktury firmy, w której pracuję. W większości przypadków nazwa działu/departamentu będzie wystarczająca.

<span style="text-decoration: underline;"><Stage></span>: Tutaj dopuszczamy dwie opcje Prod albo NonProd. Do subskrypcji produkcyjnych wpuszczamy osoby upoważnione, w tych nieprodukcyjnych oddajemy &#8220;sterowanie&#8221; liderom projektów i osobom spoza ścisłego, zaufanego grona. W NonProdach używamy innego i tańszego typu subskrypcji (a konkretnie Dev/Test), możemy wdrożyć ARM Policies na typ/lokalizację/tier zasobów. Bez obaw możemy dodać dewelopera jako co-admina w celu umożliwienia dostępu do starego portalu (na szczęście coraz rzadziej) &#8211; bez podziału na Prod/NonProd uzyskałby on dostęp do produkcji.

### Standardy nazewnicze zasobów w subskrypcji

Używany przeze mnie format w przypadku grup zasobów i samych zasobów jest nieco zbliżony do powyższego, dotyczącego samych subskrypcji: <code class="EnlighterJSRAW" data-enlighter-language="null">&lt;ProjectName&gt;-&lt;Stage&gt;-&lt;Location&gt;-&lt;ResourceType&gt;-&lt;Description&gt;</code>

<span style="text-decoration: underline;"><ProjectName></span>: Nazwa projektu, do którego należy dany zasób. Często pokrywa się z nazwą projektu w nazwie subskrypcji.

<span style="text-decoration: underline;"><Stage></span>: Zdefiniowany wcześniej słownik skrótów, które reprezentują typ środowiska. Może się tu pojawić Prod, Test, Dev, Uat, Hotfix itp.

<span style="text-decoration: underline;"><Location></span>: Trzyliterowy skrót słownikowy (czasem z dodatkową cyfrą) opisujący region, w którym znajduje się zasób. Dla Europe West &#8220;euw&#8221;, US East 2 &#8220;use2&#8221; itp.

<span style="text-decoration: underline;"><ResourceType></span>: W tym miejscu również proponuję słownik, który reprezentuje typ zasobu. Dla grupy zasobów może to być &#8220;rg&#8221;, dla sieci wirtualnej &#8220;vnet&#8221;, dla AppService Plan &#8220;plan&#8221; itp.

<span style="text-decoration: underline;"><Description></span>: Skrót opisujący cel istnienia zasobu, umożliwia on odróżnienie zasobów tego samego typu w jednej subskrypcji.

Dla przykładu, ten blog mógłby składać się z następujących zasobów:

  * personalweb-prod-eun-webapp-wphosting:  **WebApp**, umiejscowiony w **North Europe**, odpowiedzialny za **produkcyjny** hosting **WordPress&#8217;a** , będący częścią projektu **personalweb**.
  * personalweb-dev-eun-appins-wphoting: **Application Insights**, umiejscowione w **North Europe**, zbierający dane z **deweloperskiego WordPress&#8217;a**, będący częścią projektu **personalweb**.

### Podsumowanie i drobne uwagi

Każda reguła ma często swoje odstępstwa, pojawią się one również tutaj.

  * Pierwszym problemem jaki możesz napotkać jest brak możliwości określenia jednego z wymaganych sufiksów. Przykładem może być Traffic Manager, który w polu Location będzie miał &#8220;Global&#8221; &#8211; inne tego typu problemy pozostawiam Tobie, da się je spokojnie rozwiązać.
  * Drugim, poważniejszym wyzwaniem są <a href="https://docs.microsoft.com/en-us/azure/architecture/best-practices/naming-conventions#naming-rules-and-restrictions" target="_blank" rel="noopener">ograniczenia</a> dla nazw poszczególnych typów zasobów w samym Azure. Największą zakałą jest tu Storage Account (przez to również Data Lake), który nie dopuszcza myślników w nazwie. Pomimo, że jego nazwa wewnętrznie jest zgodna z RFC 1123, grupa produktowa celowo zablokowała możliwość używania tego znaku &#8220;zwykłym&#8221; użytkownikom. Ze względu na swoją wygodę trochę utrudnili nam życie i nie planują tego zmienić w przewidywalnej przyszłości! Na pocieszenie mamy &#8220;-secondary&#8221; z myślnikiem w opcji RA-GRS 🙂
  * Byłoby świetnie, gdyby tego typu konwencję dało się wdrożyć przy pomocy Azure Resource Policies. Niestety są one w chwili obecnej zbyt ograniczone, by dało się te reguły przy ich pomocy wyrazić. Kolejna moja dyskusja z grupą produktową zaowocowała propozycją, by dodali do zestawu <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-policy#policy-rule" target="_blank" rel="noopener">dostępnych Conditions</a> opcję RegEx. Czekam aż mój pomysł kiedyś dotrze na szczyt ich backlogu. Może skorzystać z UserVoice?!

Mam nadzieję, że przedstawione powyżej standardy nazewnicze pomogą Ci w przyszłości skutecznie zapanować nad tworzonymi zasobami w Azure. **Jeżeli masz już wdrożony standard, napisz proszę w komentarzach jak on wygląda!** Może uda się nam połączyć wszystko co najlepsze i stworzyć coś naprawdę unikalnego i wygodnego w użyciu.