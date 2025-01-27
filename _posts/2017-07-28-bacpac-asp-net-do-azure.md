---
title: Wdrożenie bacpac i aplikacji ASP.NET do Azure
date: 2017-07-28T07:30:43+02:00
header:
  teaser: /assets/images/2017/07/17319736742_d5e6ecf551_z.jpg
categories:
  - Architecture
tags:
  - ARM
  - ASP.NET
  - Azure
  - Bacpac
  - GitHub
  - WebApp
---
Na początku swojej przygody z blogowaniem przeprowadziłem krótkie rozeznanie na temat dostępnych platform pod blogi. Mój pierwszy wybór padł na <a href="https://github.com/rxtur/BlogEngine.NET" target="_blank" rel="noopener">BlogEngine.NET</a>. Dość naturalne wydawało mi się, że skoro jest to platforma napisana w .NET, będę miał większą nad nią kontrolę, plus będę mógł ją z łatwością wdrożyć do Azure. Ostatecznie okazało się, że ilość dostępnych dodatków i rozszerzeń nie jest zadowalająca i stanęło na WordPressie. Zanim jednak przesiadłem się na WP, dostosowałem BlogEngine.NET pod chmurę. Poniżej znajdziesz opis tego procesu, da się go pewnie zastosować przy przenoszeniu innych aplikacji ASP.NET do Azure.
{: style="text-align: justify;"}

## Przygotowanie infrastruktury i szablonów ARM

Pierwszym krokiem, który wykonałem, było zastanowienie się, jakich komponentów Azure mój blog potrzebuje. Wybór padł rzecz jasna na usługi w modelu platformowym, czyli PaaS. Sama aplikacja miała wylądować w <a href="https://azure.microsoft.com/en-us/services/app-service/web/" target="_blank" rel="noopener">WebApps </a>na AppService, baza danych w <a href="https://azure.microsoft.com/en-us/services/sql-database/" target="_blank" rel="noopener">SQL Database</a>, a logowanie wszelkich błędów i danych o wydajności w <a href="https://azure.microsoft.com/en-us/services/application-insights/" target="_blank" rel="noopener">Application Insights</a>. Dla osób, które taką konfigurację widziały już wielokrotnie, obiecuję jeden trick (bacpac) w dalszej części, którego się przy okazji nauczyłem.
{: style="text-align: justify;"}

![img](/assets/images/2017/07/BlogEngine.png)

Kolejnym krokiem jest przygotowanie szablonu ARM, by nasza architekturę każdorazowo wdrażać automatycznie. Jest to szczególnie ważne, ponieważ otwiera nam to spektrum możliwości i korzyści. Jeżeli temat nie jest Ci bliski, to odsyłam do następujących tematów: Continuous Delivery, Continuous Configuration Automation i Infrastructure as a Code.
{: style="text-align: justify;"}

W tym celu zrobiłem fork oficjalnego repozytorium BlogEngine (ostateczny wynik z poprawkami opisanymi poniżej znajdziesz <a href="https://github.com/mgrabarz/BlogEngine.NET" target="_blank" rel="noopener">tutaj</a>). Do solucji projektu dodałem nowy projekt typu Azure Resource Group z poziomu VisualStudio, natomiast można się bez tego obyć. Sam szablon json, umieszczony luzem poza projektem,  jest w zupełności wystarczający.
{: style="text-align: justify;"}

![img](/assets/images/2017/07/New-Project.png)

Od tego momentu możemy zacząć opisywać naszą docelową infrastrukturę w pliku i automatycznie wdrażać ją do Azure. Sam proces wdrażania można wykonać testowo <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/vs-azure-tools-resource-groups-deployment-projects-create-deploy" target="_blank" rel="noopener">bezpośrednio z VS</a>, lub na wiele innych sposobów, np. przez <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-template-deploy" target="_blank" rel="noopener">PowerShell</a>. Osobiście rekomenduję, by zbudować pełny proces w oparciu o VSTS, ale w opisywanym przypadku sytuacja wygląda nieco inaczej ze względu na potrzeby. Pisanie szablonów przy pierwszym zetknięciu może być nieco skomplikowane, na szczęście VS trochę nam sprawę ułatwia. Możemy również skorzystać z gotowców dostępnych na <a href="https://github.com/Azure/azure-quickstart-templates" target="_blank" rel="noopener">Azure Quickstart Templates</a>.
{: style="text-align: justify;"}

## Zawartość szablonu ARM

Szablon zawiera zwykle trzy sekcje. Pierwsza z nich służy do definicji parametrów wejściowych (w naszym przypadku mogą to być login i hasło do bazy, adres bloga, półka cenowa usług itp). Kolejna sekcja to opis infrastruktury docelowej, wykorzystujący parametry. W ostatniej części znajdują się deklaracje parametrów wyjściowych z procesu.
{: style="text-align: justify;"}

W moim szablonie definiuję dokładnie to, co zobrazowałem na diagramie w poprzednim akapicie. Poniżej deklaracja Application Insights, na uwagę zasługuje użycie wspomnianych parametrów i chyba jedna z ważniejszych spraw - "dependsOn", czyli wskazanie co najpierw musi być stworzone, by stworzenie tego kawałka się powiodło.
{: style="text-align: justify;"}

```json
/*Application Insights */
{
      "apiVersion": "2014-04-01",
      "name": "[parameters('applicationInsightsName')]",
      "type": "Microsoft.Insights/components",
      "location": "westeurope",
      "dependsOn": [
        "[resourceId('Microsoft.Web/sites/', parameters('websiteName'))]"
      ],
      "tags": {
        "[concat('hidden-link:', resourceGroup().id, '/providers/Microsoft.Web/sites/', parameters('websiteName'))]": "Resource",
        "displayName": "AppInsightsComponent"
      },
      "properties": {
        "ApplicationId": "[parameters('websiteName')]"
      }
    }
```

W podobny sposób deklarowana jest WebApp (która wymaga swojego hosting planu).
{: style="text-align: justify;"}

```json
{
      "apiVersion": "2015-08-01",
      "name": "[parameters('websiteHostingPlanName')]",
      "type": "Microsoft.Web/serverfarms",
      "location": "[resourceGroup().location]",
      "tags": {
        "displayName": "HostingPlan"
      },
      "sku": {
        "name": "[parameters('websitePricingTier')]",
        "capacity": "[parameters('websiteInstanceNodeCount')]"
      },
      "properties": {
        "name": "[parameters('websiteHostingPlanName')]"
      }
    },
    {
      "apiVersion": "2015-08-01",
      "name": "[parameters('websiteName')]",
      "type": "Microsoft.Web/sites",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Web/serverFarms/', parameters('websiteHostingPlanName'))]"
      ],
      "tags": {
        "[concat('hidden-related:', resourceGroup().id, '/providers/Microsoft.Web/serverfarms/', parameters('websiteHostingPlanName'))]": "empty",
        "displayName": "Website"
      },
      "properties": {
        "name": "[parameters('websiteName')]",
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', parameters('websiteHostingPlanName'))]"
      },
      "resources": [
        {
          "apiVersion": "2015-08-01",
          "type": "config",
          "name": "connectionstrings",
          "dependsOn": [
            "[resourceId('Microsoft.Web/Sites/', parameters('websiteName'))]",
            "[resourceId('Microsoft.Web/Sites/sourcecontrols', parameters('websiteName'), 'web')]"
          ],
          "properties": {
            "DefaultConnection": {
              "value": "[concat('Data Source=tcp:', reference(resourceId('Microsoft.Sql/servers/', parameters('sqlServerName'))).fullyQualifiedDomainName, ',1433;Initial Catalog=', parameters('databaseName'), ';User Id=', parameters('databaseAdministratorLogin'), '@', parameters('sqlServerName'), ';Password=', parameters('databaseAdministratorPassword'), ';')]",
              "type": "SQLServer"
            },
            "BlogEngine": {
              "value": "[concat('Data Source=tcp:', reference(resourceId('Microsoft.Sql/servers/', parameters('sqlServerName'))).fullyQualifiedDomainName, ',1433;Initial Catalog=', parameters('databaseName'), ';User Id=', parameters('databaseAdministratorLogin'), '@', parameters('sqlServerName'), ';Password=', parameters('databaseAdministratorPassword'), ';')]",
              "type": "SQLServer"
            }
          }
        },
        {
          "apiVersion": "2015-08-01",
          "type": "config",
          "name": "appsettings",
          "dependsOn": [
            "[resourceId('Microsoft.Web/Sites/', parameters('websiteName'))]",
            "[resourceId('Microsoft.Insights/components/', parameters('applicationInsightsName'))]",
            "[resourceId('Microsoft.Web/Sites/sourcecontrols', parameters('websiteName'), 'web')]"
          ],
          "properties": {
            "applicationInsightsInstrumentationKey": "[reference(resourceId('Microsoft.Insights/components', parameters('applicationInsightsName')), '2014-04-01').InstrumentationKey]"
          }
        },
        {
          "apiVersion": "2015-08-01",
          "name": "web",
          "type": "sourcecontrols",
          "dependsOn": [
            "[resourceId('Microsoft.Web/Sites', parameters('websiteName'))]"
          ],
          "properties": {
            "RepoUrl": "https://github.com/mgrabarz/BlogEngine.NET.git",
            "branch": "master",
            "IsManualIntegration": true
          }
        }
      ]
    },
```

Podczas tworzenie WebApp deklaruję elementy zagnieżdżone, jak **appSettings** lub **connectionStrings**. Mają one inne **dependsOn**, ponieważ ich wartości będą znane po utworzeniu innych zasobów (np parametry połączenia do bazy danych). Jedną z ciekawostek jest podsekcja **sourcecontrols**. Przy jej pomocy, po wdrożeniu infrastruktury **z GitHub zaciągany jest kod aplikacji, kompilowany/testowany w locie i wgrywany do chmury**. Wszystko w zależności od zadeklarowanych ustawień.
{: style="text-align: justify;"}

Identycznie wdrażana jest baza danych oraz reguły monitorujące i alerty &#8211; które też można deklarować w szablonach. Cały szablon znajdziesz <a href="https://github.com/mgrabarz/BlogEngine.NET/blob/master/BlogEngine/BlogEngine.ArmDeployment/WebSiteSQLDatabase.json" target="_blank" rel="noopener">tutaj</a>, powinien on wystarczyć do wdrożenia większości nieskomplikowanych aplikacji ASP.NET do Azure.
{: style="text-align: justify;"}

## Dodatkowe zmiany w aplikacji

Po wdrożeniu bazy w Azure doczytałem, że BlogEngine domyślnie przechowuje dane bloga w plikach na dysku. W samym Azure WebApp takie podejście by przeszło, powodowało nieco problemów szczególnie przy dużych plikach, nie gwarantowało wszystkich cech ACID transakcji. Spytacie dlaczego, otóż w chmurze mamy skalowanie, dane plikowe się replikują na poszczególne nogi naszego rozwiązania, czasami niektóre serwery się składają, a usługa podstawia nowe.
{: style="text-align: justify;"}

Zdecydowałem się na pójście drugą z możliwych opcji, czego sygnałem wcześniej jest stworzenie bazy w szablonie ARM. Wykonałem <a href="https://github.com/rxtur/BlogEngine.NET/tree/master/BlogEngine/BlogEngine.NET/setup/SQLServer" target="_blank" rel="noopener">następujące </a>zmiany w konfiguracji aplikacji i uświadomiłem sobie, że baza nie może być pusta tuż po wdrożeniu. Przy pełnym cyklu wdrożeniowym z VSTS nie stanowiłoby to problemu, ja chciałem zostawić jedynie repozytorium na GitHub. Z pomocą przyszła mi technika wdrażania plików bacpac bezpośrednio w szablonie. **Tak jak bacpac używałem od dawna, tak nie spodziewałem się ich obsługi w ARM**.
{: style="text-align: justify;"}

Czym są pliki bacpac? Można powiedzieć, że czymś zbliżonym do backupu bazy (struktura plus dane), z kolei pliki dacpac zawierają jedynie strukturę. Na tym różnice się nie kończą. Plik backupu traktowany jest jak czarne pudełko, nie możemy z nim niewiele zrobić poza odtworzeniem. Pliki bacpac/dacpac możemy używać w celu porównywania schematów bazy między sobą, a nawet wdrożenia różnicowego. I tak, możemy wdrożyć różnice między plikiem a bazą, między projektem bazy (w VisualStudio) a bazą, między projektem a plikiem itp. Więcej informacji na temat Data-Tier Applications znajdziesz <a href="https://docs.microsoft.com/en-us/sql/relational-databases/data-tier-applications/data-tier-applications" target="_blank" rel="noopener">tutaj</a>.
{: style="text-align: justify;"}

W skrócie skonfigurowałem lokalną bazę, wgrałem potrzebne dane, zrzuciłem ją do bacpac&#8217;a, ten z kolei umieściłem w publicznie dostępnym miejscu. W szablonie, jako podelement szablonu bazy, dokonuję jej importu, zgodnie z zwartością bacpac:
{: style="text-align: justify;"}

```json
"resources": [
            {
              "type": "extensions",
              "apiVersion": "2014-04-01",
              "properties": {
                "operationMode": "Import",
                "storageKey": "?",
                "storageKeyType": "SharedAccessKey",
                "administratorLogin": "[parameters('databaseAdministratorLogin')]",
                "administratorLoginPassword": "[parameters('databaseAdministratorPassword')]",
                "storageUri": "https://blogenginegithub.blob.core.windows.net/database/database.bacpac"
              },
              "name": "Import",
              "dependsOn": [
                "[resourceId('Microsoft.Sql/servers/databases', parameters('sqlServerName'), parameters('databaseName'))]"
              ]
            }
          ]
        }
```

Ostatnim krokiem przed wgraniem naszej aplikacji ASP.NET do Azure było skonfigurowanie ApplicationInsights w jej kodzie. W szablonie ARM propagowany jest do &#8220;application settings&#8221; klucz telemetryczny AI. AI tworzy się w pierwszej kolejności, a &#8220;dependsOn&#8221; settingów czeka, aż klucz będzie możliwy do pobrania. Proces konfiguracji w kodzie nieco się może różnić w moim przypadku, więc nie będę go tu opisywał, ale dobre wytyczne można znaleźć <a href="https://docs.microsoft.com/pl-pl/azure/application-insights/app-insights-asp-net" target="_blank" rel="noopener">tutaj</a>.
{: style="text-align: justify;"}

## Podsumowanie

Powyższy proces, jak już wspominałem, powinien być zautomatyzowany w narzędziu takim jak VSTS, TeamCity czy Jenkins. Moim celem było skonfigurowanie wdrożenia w taki sposób, by jednym kliknięciem umieścić rozwiązanie w Azure. Poprzez użycie szablonów ARM, w tym deploymentu git i bacpac, cel został zrealizowany.
{: style="text-align: justify;"}

Jeżeli chcesz wrzucić BlogEngine.NET do swojej subskrypcji Azure, śmiało klikaj w poniższe guziki.
{: style="text-align: justify;"}

[Deploy](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmgrabarz%2FBlogEngine.NET%2Fmaster%2FBlogEngine%2FBlogEngine.ArmDeployment%2FWebSiteSQLDatabase.json)

[ARM](http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2Fmgrabarz%2FBlogEngine.NET%2Fmaster%2FBlogEngine%2FBlogEngine.ArmDeployment%2FWebSiteSQLDatabase.json)

Photo credit: [john farrell macdonald](https://www.flickr.com/photos/jfmacdonald/17319736742/) via [Visual Hunt](https://visualhunt.com/re/35b4b5) /  [CC BY-SA](http://creativecommons.org/licenses/by-sa/2.0/)
{: style="text-align: justify;"}