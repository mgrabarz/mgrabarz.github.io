---
title: Automatyzacja użycia Function Key w Azure Functions
date: 2017-03-26T10:55:42+02:00
header:
  teaser: /assets/images/2017/04/07-04-2017-functions1.png
categories:
  - Functions
tags:
  - Azure
  - DevOps
  - Functions
---
## Azure Functions, uwierzytelnianie i autoryzacja

Podczas implementacji funkcji z HTTP trigger musimy pamiętać, że jest ona wystawiona do Internetu i każdy, kto zna jej adres może ją wykonać. Najprostszym sposobem zabezpieczenia dostępu do takiej funkcji jest użycie Function Keys, pisał o nich między innymi Łukasz Kałużny na swoim [blogu](http://blog.kaluzny.pro/azure-functions-autoryzacja-za-pomoca-kluczy/).

W dużym skrócie rozróżniamy dwa klucze, Host Key definiowany na poziomie całego FunctionApp oraz Function Key dedykowany dla jednej funkcji. Uwierzytelnianie kluczem odbywa się na jeden z dwóch wspieranych sposobów:

  * Klucz jest przekazywany jako parametr w QueryStringu, np: <code class="EnlighterJSRAW" data-enlighter-language="html">&lt;yourapp&gt;.azurewebsites.net/api/&lt;funcName&gt;?code=&lt;yourApiKey&gt;</code>
  * Klucz jest przekazywany w nagłówku żądania: <code class="EnlighterJSRAW" data-enlighter-language="html">x-functions-key</code>

Po zweryfikowaniu klucza przechodzimy do etapu autoryzacji, tutaj mamy trzy możliwości skonfigurowania naszej funkcji:

  * Anonymous, dopuszcza każde żądanie, w tym również te bez klucza
  * Function, dopuszcza tylko uwierzytelnione żądania z Host Key lub z Function Key
  * Admin, dopuszcza tylko uwierzytelnione żądania z Host Key.

Kluczami możemy zarządzać poprzez portal, lub przy pomocy API, przy czym w dokumentacji znajdziemy następujący, nieco rozczarowujący wpis:

_&#8220;To support programmatic key management, the host exposes a key management API. API documentation coming soon.&#8221;_

## Automatyzacja i zarządzanie kluczami w Azure Functions

Wspomniane powyżej zarządzanie kluczami w portalu jest stosunkowo proste, ale wymaga ono ręcznej interwencji, szczególnie w przypadku nowo utworzonych funkcji. Moim celem z kolei jest wyeliminowanie jakichkolwiek ręcznych prac (między innymi w celu obniżenia kosztów obsługi oraz obniżenia ryzyka błędów).

W moim VSTS, na którym przechowuję kod rozwiązania, definiuję Build Definition oraz Release Definition. Mój kod jest zweryfikowany, następuje wdrożenie szablonu ARM z funkcją wraz z zależnymi od niej elementami i pojawia się problem: Jak uzyskać Function Key funkcji, która właśnie powstała, by go następnie automatycznie wstrzyknąć do AppSettings i ConnectionStrings innych rozwiązań?

Pierwszy pomysł to próba zdefiniowania klucza w szablonie ARM, lub uzyskanie go jako output z szablonu. Niestety okazuje się, że obecnie nie jest to wspierane, dodatkowo na próżno szukać kluczy w [resources.azure.com](https://resources.azure.com), po prostu ich tam nie ma. Klucz przechowywany jest na dyskach FunctionApp, a konkretnie w następującym pliku (dostępnym przez FTP lub Kudu)

/data/Functions/secrets/host.json &#8211; klucze Host /data/Functions/secrets/{nazwaFunkcji}.json &#8211; klucze Function

Pliki są zaszyfrowane, można je zastąpić niezaszyfrowanymi wersjami ze zdefiniowanymi przez nas kluczami. Rozwiązanie takie, mimo że wykonalne, nie zalicza się do najbardziej eleganckich  &#8211; zdecydowanie nie zasługuje na tak długi opis. Na szczęście pozostaje nam jeszcze API, a konkretnie Kudu API:

  1. Wysyłamy żądanie GET na adres Kudu, w celu uzyskania Master Key. <code class="EnlighterJSRAW" data-enlighter-language="html">&lt;yourApp&gt;.scm.azurewebsites.net/api/functions/admin/masterkey</code>_<span style="color: #3366ff;"><img class="alignnone size-full wp-image-181" src="http://marek.grabarze.com/wp-content/uploads/2017/03/kudu_masterkey1.png" alt="Kudu_MasterKey" width="712" height="148" srcset="https://marekgrabarz.pl/wp-content/uploads/2017/03/kudu_masterkey1.png 712w, https://marekgrabarz.pl/wp-content/uploads/2017/03/kudu_masterkey1-300x62.png 300w" sizes="(max-width: 712px) 100vw, 712px" /></span>_
  2. Klucz Master Key umożliwia pozyskanie kluczy zarówno Host Key, jak również Function Key dla poszczególnych funkcji.
  3. W celu uzyskania Function Key, wysyłamy GET na adres: <code class="EnlighterJSRAW" data-enlighter-language="html">&lt;yourApp&gt;.azurewebsites.net/admin/functions/&lt;functionName&gt;/KEYS?CODE=&lt;masterKey&gt;</code><span style="color: #3366ff;"><em> <img class="alignnone size-full wp-image-179" src="http://marek.grabarze.com/wp-content/uploads/2017/03/kudu_funckey.png" alt="Kudu_FuncKey" width="1150" height="315" srcset="https://marekgrabarz.pl/wp-content/uploads/2017/03/kudu_funckey.png 1150w, https://marekgrabarz.pl/wp-content/uploads/2017/03/kudu_funckey-300x82.png 300w, https://marekgrabarz.pl/wp-content/uploads/2017/03/kudu_funckey-768x210.png 768w, https://marekgrabarz.pl/wp-content/uploads/2017/03/kudu_funckey-1024x280.png 1024w" sizes="(max-width: 1150px) 100vw, 1150px" /></em></span>
  4. Pobranie Host Key odbywa się poprzez wysłanie GET na adres: <code class="EnlighterJSRAW" data-enlighter-language="html">&lt;yourApp&gt;.azurewebsites.net/admin/HOST/KEYS?CODE=&lt;masterKey&gt;</code><span style="text-decoration: underline;"><span style="color: #3366ff; text-decoration: underline;"><em><img class="alignnone size-full wp-image-180" src="http://marek.grabarze.com/wp-content/uploads/2017/03/kudu_hostkey.png" alt="Kudu_HostKey" width="1024" height="322" srcset="https://marekgrabarz.pl/wp-content/uploads/2017/03/kudu_hostkey.png 1024w, https://marekgrabarz.pl/wp-content/uploads/2017/03/kudu_hostkey-300x94.png 300w, https://marekgrabarz.pl/wp-content/uploads/2017/03/kudu_hostkey-768x242.png 768w" sizes="(max-width: 1024px) 100vw, 1024px" /></em></span></span>

## Podsumowanie

Przy użyciu powyższych wywołań można w pełni zautomatyzować wdrożenie Azure Functions i rozpropagować klucze do innych zależnych komponentów systemu. Wywołanie samego API również nie powinno stanowić wyzwania &#8211; wymagany jest jedynie odpowiedni poziom uprawnień dla Service Principal reprezentujący VSTSa.

Daj mi proszę znać w komentarzach, czy powyższy artykuł jest dla ciebie przydatny lub ciekawy. Może znasz inny sposób na zarządzanie kluczami w Azure Functions?

_EDIT 21.04.2017 (dzięki Kamil za celne spostrzeżenia)._

_Wywołanie wspomnianych powyżej API rzeczywiście może przysporzyć nieco problemów._ 

_Kudu API wymaga deployment credentials, które możemy pobrać z naszej FunctionApp, przykład <a href="http://stackoverflow.com/questions/27443368/azure-websites-kudu-rest-api-authentication" target="_blank" rel="noopener noreferrer">tutaj</a>._

_Drugie wywołanie z kolei wymaga klasycznego Bearer tokena, przykład jego generowania w Powershell wygląda mniej więcej tak:_

<pre class="EnlighterJSRAW"># Load ADAL Assemblies
$adal = "C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ResourceManager\AzureResourceManager\AzureRM.Profile\Microsoft.IdentityModel.Clients.ActiveDirectory.dll"
$adalforms = "C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ResourceManager\AzureResourceManager\AzureRM.Profile\Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms.dll"
[System.Reflection.Assembly]::LoadFrom($adal)
[System.Reflection.Assembly]::LoadFrom($adalforms)
# Set Azure AD Tenant name
$adTenant = "YOUR_NAME.onmicrosoft.com" 
# Set well-known client ID for AzurePowerShell
$clientId = "1950a258-227b-4e31-a9cf-717495945fc2" 
# Set redirect URI for Azure PowerShell
$redirectUri = "urn:ietf:wg:oauth:2.0:oob"
# Set Resource App URI as ARM
$resourceAppIdURI = "https://management.azure.com/"
# Set Authority to Azure AD Tenant
$authority = "https://login.windows.net/$adTenant"
# Create Authentication Context tied to Azure AD Tenant
$authContext = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext" -ArgumentList $authority
# Acquire token
$authResult = $authContext.AcquireToken($resourceAppIdURI, $clientId, $redirectUri, "never")
# Output bearer token
$authHeader = $authResult.CreateAuthorizationHeader()


# Get the host key
$hostKeyRequest = Invoke-RestMethod -Method GET -Uri "https://APP_NAME/admin/HOST/KEYS?CODE=MASTER_KEY -Headers @{ Authorization = $authHeader }</pre>