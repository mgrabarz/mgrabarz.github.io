---
title: API Management - Konfiguracja security
date: 2017-10-07T16:19:18+02:00
header:
  teaser: /assets/images/2017/10/frozen-locked-door-with-blue-padlock.jpg
categories:
  - APIM
tags:
  - API
  - API Management
  - APIM
  - RBAC
  - Security
  - Governance
---
W zeszłym tygodniu <a href="https://grabarz.pl/apim/azure-api-management/" target="_blank" rel="noopener">rozpocząłem </a>serię wpisów na temat Azure API Management. W dzisiejszej części poruszę tematykę  modelu uprawnień wbudowanego w usługę. Pomimo że na pierwszy rzut oka wydaje się on bardzo prosty, po przeczytaniu poniższego wpisu będziesz mógł w pełni kontrolować uprawnienia do zarządzania treścią widoczną na portalu.
{: style="text-align: justify;"}

## Publisher Portal i Developer Portal

Czytając kiedyś o Azure API Management trafiłem na pewną historię. Podobno usługa ta nie była od samego początku planowana w Azure. Jeden z zespołów w Microsoft zwyczajnie stworzył jej podstawową wersję na potrzeby swoich projektów, a ta została włączona z czasem do usług dostępnych dla klientów. Nie wiem ile jest w tym prawdy, ale można odnieść wrażenie, że coś w tym jest patrząc na Portale w APIM. Jak na usługę za górę pieniędzy są one delikatnie mówiąc minimalistyczne i niestety to samo można powiedzieć o starym modelu uprawnień.
{: style="text-align: justify;"}

Bazujemy na trzech podstawowych grupach uprawnień. Nie możemy ich modyfikować, ani bezpośrednio zarządzać członkowstwem w tych grupach.
{: style="text-align: justify;"}

- **Guests** &#8211; Osoby nieuwierzytelnione, anonimowi użytkownicy Developer Portalu. Przy odpowiednich ustawieniach mają prawo czytać dokumentację publicznych API naszej firmy.
- **Developers** &#8211; Osoby które zostały uwierzytelnione do Developer Portalu przez którykolwiek z mechanizmów logowania skonfigurowany w usłudze. Mają wgląd (dokumentacja/ użycie) do wszystkich opublikowanych API i Produktów (patrz integracja z AD poniżej).
- **Administrators** &#8211; Ludzie mający dostęp do API Management z poziomu portalu Azure. Minimalne wymaganie to Contributor lub **Azure API Management Service Contributor** w RBAC. Administratorzy jako jedyni mają dostęp do Publisher Portalu.

Powyżej pisałem o Administratorach, ale mi osobiście nie udało się nigdy widzieć dwóch osób w tej grupie. Każda osoba z prawami w RBAC po prostu wchodzi na Publisher Portal i jest domyślnie wkładana do jednego konta-worka o nazwie **Administrator**. Zabawne jest to, że tenże wspólny Administrator, jeżeli stworzyliśmy APIM bez zmiany domyślnych ustawień, ma maila twórcy. To z kolei sprawia, że nie da się tego maila użyć do SignUp jako Developer (jakoś zrozumiem) i do logowania do Developer Portalu bo jest to konto typu **Azure**, a nie **Basic**, czy **AAD**. Pozostaje zmienić email Administratora, w tym pomogła mi dopiero grupa produktowa z Redmond, sugerując następujący skrypt:
{: style="text-align: justify;"}

```powershell
$Resource = Get-AzureRmResource -ResourceType "microsoft.apimanagement/service" -ResourceGroupName "<resource group name>" -ResourceName "<api management service name>" -ApiVersion "2017-03-01"

$Resource.Properties.publisherEmail = "<Administrator's email>"
$Resource.Properties.publisherName = "<Organization name, you will receive notifications from>"
$Resource.Properties.notificationSenderEmail="<Notifications email>"

$Resource | Set-AzureRmResource -Force -ApiVersion "2017-03-01"
```

Sytuację ratuje integracja z Azure AD. Możemy dodawać nowe **grupy pochodzące z AAD** i w oparciu o nie zmieniać widoczność niektórych bardziej "tajnych" API. W efekcie dla zwykłych Developers niektóre API/Produkty w Developer Portalu nie będą widoczne, ani nie będą oni w stanie uzyskać do nich dostępu.
{: style="text-align: justify;"}

## Rozszerzone uprawnienia RBAC w API Management

W związku z faktem, że API Management aktualnie migruje się z Publisher Portalu do portalu Azure, mamy możliwość spróbowania nowych uprawnień w modelu RBAC. Oczywiście nie dotyczą one bezpośrednio Publisher Portalu, tam zwyczajnie mamy pełen dostęp, lub nie mamy go wcale. Poniżej zestawienie ról wbudowanych:
{: style="text-align: justify;"}

- **Reader** tożsamy z **Azure API Management Service Reader** - Uprawnienia do odczytu konfiguracji usługi w portalu Azure, brak dostępu do Publisher Portal.
- **Contributor/Owner **tożsamy z** Azure API Management Service Contributor** - Pełna administracja usługą w portalu Azure i przez Publisher Portal.
- **Azure API Management Service Operator** - Uprawnienia do zarządzania usługą, bez możliwości edycji API, produktów i bez dostępu do Publisher Portal.
- W planach są również dwie nowe role wbudowane, **Azure API Management Service Editor** oraz **Azure API Management Content Manager**. Zostaną one upublicznione po przeniesieniu Publisher Portalu do portalu Azure. Pierwsza umożliwi edycję API, druga zarządzanie Developer Portalem i wraz rolą Operatora stanowić będą całość uprawnień Contributora.
{: style="text-align: justify;"}

Poza powyższymi rolami mamy możliwość tworzenia własnych ról RBAC. W kilka chwil możemy stworzyć rolę, która umożliwia np edycję wybranego API, lub edycję tylko polityk na wybranej metodzie API. Ilość możliwych uprawnień z których skomponujemy rolę jest <a href="https://docs.microsoft.com/en-us/azure/active-directory/role-based-access-control-resource-provider-operations#microsoftapimanagement" target="_blank" rel="noopener">spora</a>, a do tego dochodzi fakt, że API, jego metoda i ich polityki są zagnieżdżonymi zasobami w ARM.
{: style="text-align: justify;"}

Poniższy skrypt demonstruje stworzenie własnej roli do edycji wskazanego API, wraz z politykami, metodami i ich politykami.
{: style="text-align: justify;"}

```powershell
$role = Get-AzureRmRoleDefinition "API Management Service Reader Role"
$role.Id = $null
$role.Name = 'My API Contributor'
$role.Description = 'Has read access to APIM instance and write access to My API.'

$role.Actions.Add('Microsoft.ApiManagement/service/apis/write')
$role.Actions.Add('Microsoft.ApiManagement/service/apis/operations/write')
$role.Actions.Add('Microsoft.ApiManagement/service/apis/operations/policy/write')
$role.Actions.Add('Microsoft.ApiManagement/service/apis/policy/write')

$role.AssignableScopes.Clear()
$role.AssignableScopes.Add('/subscriptions/<subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.ApiManagement/service/<service name>/apis/<api ID>')

New-AzureRmRoleDefinition -Role $role
```

Po utworzeniu roli możemy ją przypisać do użytkownika
{: style="text-align: justify;"}

```powershell
New-AzureRmRoleAssignment -ObjectId <object ID of the user or group> -RoleDefinitionName 'My API Contributor' -Scope '/subscriptions/<subscription ID>/resourceGroups/<resource group name>/providers/Microsoft.ApiManagement/service/<service name>/apis/<api ID>'
```

Jedyna trudność to znalezienie API ID, lub całego Scope w uprawnieniu. W tym celu, poza metodami skryptowymi, zawsze można wejść na <a href="https://resources.azure.com" target="_blank" rel="noopener">resources.azure.com</a>, tam zlokalizować nasz API Management, konkretne API i skopiować jego adres.
{: style="text-align: justify;"}

## Następne kroki

Mam nadzieję, że dzisiejszy wpis Cię zainteresował. W następnym omówimy możliwości integracji logowania do Developer Portalu z AAD, AAD B2C i innymi typami tożsamości.
{: style="text-align: justify;"}

Jeżeli interesuje Cię jakiś konkretny temat związany z APIM i chcesz, bym go opisał, daj koniecznie znać.
{: style="text-align: justify;"}

Photo via [Visualhunt](https://visualhunt.com/re/c751e5)
{: style="text-align: justify;"}
