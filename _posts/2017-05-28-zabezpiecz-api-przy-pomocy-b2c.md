---
title: Zabezpiecz swoje API przy pomocy B2C część druga
date: 2017-05-28T23:55:46+02:00
header:
  teaser: /assets/images/2017/05/computer-keyboard.jpg
categories:
  - B2C
tags:
  - AspNetCore1.1
  - B2C
  - GitHub
  - MSAL
  - OAuth2.0
  - OpenIdConnect
---
W poprzednim [wpisie](/b2c/azure-b2c-access-tokens/) obiecałem, że rozwinę temat zabezpieczania API przy pomocy Azure AD B2C. Od tego czasu na blogu nic się nie działo i wynikało to z kilku powodów:

- Wpis ten wymagał stworzenia działającego przykładu, a to z kolei wymagało nieco więcej czasu.
- W międzyczasie wpadło kilka innych tematów do kalendarza (Microsoft Build 2017, wyjazdy służbowe, prezentacja na MAUGP, majówka).
- Komponenty używane w przykładzie, w szczególności <a href="https://github.com/AzureAD/microsoft-authentication-library-for-dotnet" target="_blank" rel="noopener noreferrer">MSAL (Microsoft Authentication Library), </a>zostały opublikowane dopiero na wyżej wspomnianym Buildzie.
- Stworzyłem jedyny znany Googlowi przykład używający MSAL, OpenIdConnect, B2C i Authorization Code Grant opierający się na .NET Core 1.1

Pełny kod źródłowy poniżej opisanego przykładu można znaleźć na moim Gitubie, <a href="https://github.com/mgrabarz/AzureAdB2C-API-AuthorizationWithScopes" target="_blank" rel="noopener noreferrer">AzureAdB2C-API-AuthorizationWithScopes</a>. Tutaj postaram się pokazać krok po kroku jak powyższy przykład zastosować z realnych rozwiązaniach.
{: style="text-align: justify;"}

### Co zostało zaimplementowane w przykładzie?

Przykładowy kod składa się z dwóch głównych komponentów, części serwerowej i części klienckiej.

Część serwerowa, wraz z mała bazą in memory, implementuje restowe API, które pozwala odczytywać, dodawać i usuwać notatki. Dodatkowo w części serwerowej API zabezpieczone jest przez B2C i wymaga pewnych dodatkowych Claim i Scope (w oparciu o OAuth2.0), które zdefiniowaliśmy w poprzednim wpisie.
{: style="text-align: justify;"}

Część kliencka to prosta strona z interfejsem webowym, która wykonuje uwierzytelnianie użytkownika przy pomocy OpenIdConnect i biblioteki MSAL, a następnie przy pomocy Authorization Code Grant flow i OAuth 2.0 wykonuje operacja na serwerowym API. W skrócie wykonuje dokładnie to, co opisałem w poprzednim wpisie w wersji z klientem prywatnym i pełnym pobraniem id_token.
{: style="text-align: justify;"}

### Część serwerowa &#8211; użycie OAuth i B2C do autoryzacji dostępu do metod API

Możemy przyjąć, że nasz startowy stan API jest taki, że metody działają, notatki są dodawane i usuwane, zaczynamy pracę nad zabezpieczeniem naszego kodu przed nieuprawnionymi użytkownikami.
{: style="text-align: justify;"}

Pierwszy krok to włączenie uwierzytelniania/autoryzacji  do naszej aplikacji i do mechanizmu przetwarzania żądań HTTP. W tym celu użyjemy dość dobrze znanego z Azure JWT/Bearer. W pliku konfiguracji aplikacji dodajemy następujący fragment:
{: style="text-align: justify;"}

```csharp
app.UseJwtBearerAuthentication(new JwtBearerOptions
{
   Authority = string.Format("https://login.microsoftonline.com/tfp/{0}/{1}",
      Configuration["Authentication:AzureAdB2C:TenantName"],
      Configuration["Authentication:AzureAdB2C:SignInPolicyName"],
   Audience = Configuration["Authentication:AzureAdB2C:ClientId"],
   MetadataAddress = string.Format(
      Configuration["Authentication:AzureAdB2C:MetadataEndpointUrlTemplate"], 
      Configuration["Authentication:AzureAdB2C:TenantName"], 
      Configuration["Authentication:AzureAdB2C:SignInPolicyName"])
});
```

Fragment ma na celu wymuszenie, by przychodzące żądania posiadały poprawny Bearer token, jego poprawność będzie sprawdzana pod katem wystawcy (Authority), oraz w jakim celu został wystawiony (Audience - identifikator aplikacji serwerowej).
{: style="text-align: justify;"}

Na kontrolerach, czy poszczególnych metodach API dodajemy atrybuty Authorize, tym razem jednak z nazwą polityki autoryzacyjnej. Tym sposobem na przykład metoda Delete naszego kontrolera wymaga spełnienia wymagań opisanych w polityce "DeleteNotes".

```csharp
// DELETE api/values/5
[HttpDelete("{id}")]
[Authorize(Policy = "DeleteNotes")]
public async Task&lt;IActionResult&gt; Delete(int id)
{ ...
```

Pozostaje nam już tylko opisać jakie wymagania maja poszczególne, zdefiniowane przez nas, polityki autoryzacyjne. W tym celu wracamy do Startup.cs i dodajemy następujący fragment kodu:
{: style="text-align: justify;"}

```csharp
//Configure authorization policies that use scopes and claims.
services.AddAuthorization(options =&gt;
{
    options.AddPolicy("ReadNotes", policy =&gt;
        policy.RequireScope(Constants.Scopes.NotesServiceReadNotesScope));

    options.AddPolicy("WriteNotes", policy =&gt;
        policy.RequireScopesAll(new[]
        {
            Constants.Scopes.NotesServiceWriteNotesScope
        })
        .RequireClaim("Name"));//We need this claim to record name of person who created note.

    options.AddPolicy("DeleteNotes", policy =&gt;
        policy.RequireScopesAll(new[]
        {
            Constants.Scopes.NotesServiceReadNotesScope,
            Constants.Scopes.NotesServiceWriteNotesScope
        })
        .RequireClaim(Constants.IdentityProviderClaim, "twitter.com"));
});
```

Powyższy fragment wydaje się dość jasny. Na uwagę zasługuje fakt, że nie znalazłem dobrej metody, która w pełni obsługuje Scope (będący Claim'em z poszczególnymi uprawnieniami oddzielonymi spacjami). W tym celu powstała klasa **ScopeAuthorizationRequirement** która takie właśnie przeszukiwanie (string.split) wykonuje oraz metody rozszerzeń do **AuthorizationPolicyBuilder**, które nasze wymaganie wstrzykują.
{: style="text-align: justify;"}

```csharp
public static class AuthorizationPolicyBuilderExtensions
    {
        public static AuthorizationPolicyBuilder RequireScope(this AuthorizationPolicyBuilder @this, string scope)
        {
            if (scope == null)
                throw new ArgumentNullException(nameof(scope));

            @this.Requirements.Add(new ScopeAuthorizationRequirement(new[] { scope }));
            return @this;
        }

        public static AuthorizationPolicyBuilder RequireScopesAll(this AuthorizationPolicyBuilder @this, IEnumerable&lt;string&gt; scopes)
        {
            foreach (var scope in scopes)
                @this.RequireScope(scope);
            return @this;
        }

        public static AuthorizationPolicyBuilder RequireScopesAny(this AuthorizationPolicyBuilder @this, IEnumerable&lt;string&gt; scopes)
        {
            @this.Requirements.Add(new ScopeAuthorizationRequirement(scopes));
            return @this;
        }
    }
```

Ostatnim, raczej opcjonalnym i zdecydowanie niepozornym fragmentem kodu jest &#8220;zawór bezpieczeństwa&#8221;. Ma on na celu domyślne wymuszenie autoryzacji na wszystkich kontrolerach, poza tymi, które jawnie oznaczymy jako ogólnodostępne. Dzięki temu zapobiegamy przypadkowemu udostępnieniu niektórych metod API dla wszystkich, bez logowania.
{: style="text-align: justify;"}

```csharp
// Removes the need for empty [Authorize] attribute
services.AddMvc(options =&gt;
{
    var requireAuthenticatedUserPolicy = new AuthorizationPolicyBuilder()
            .RequireAuthenticatedUser() 
            .Build();
    options.Filters.Add(new AuthorizeFilter(requireAuthenticatedUserPolicy));
});
```

### Część kliencka &#8211; MSAL i Authorzation Code Grant

Drugą składową prezentowanego przykładu jest część kliencka. Przysporzyła mi ona nieco więcej pracy, głównie z powodu braku przykładów użycia OpenIdConnect w .NET Core 1.1. Ci którzy używają Core na co dzień wiedzą, że wbrew swojej nazwie, znacznie różni się on od .NET Core 1.0 (nie wspominając, że przed kilkoma tygodniami pojawiło się preview wersji 2.0 &#8211; strach się bać). Przykładowe użycie w iOS, Android, czy klasycznym .NET można znaleźć na oficjalnym repo <a href="https://github.com/Azure-Samples?utf8=%E2%9C%93&q=MSAL&type=&language=" target="_blank" rel="noopener noreferrer">Azure Samples</a>.
{: style="text-align: justify;"}

Pierwszy krok to ściągnięcie prerelease MSAL z oficjalnego <a href="https://www.nuget.org/packages/Microsoft.Identity.Client/" target="_blank" rel="noopener noreferrer">NuGeta</a>, a jeżeli chcemy poeksperymentować z nowszymi wersjami, z nieoficjalnego <a href="https://www.myget.org/feed/aad-clients-nightly/package/nuget/Microsoft.Identity.Client" target="_blank" rel="noopener noreferrer">MyGeta</a>.
{: style="text-align: justify;"}

W kodzie definiujemy uwierzytelnianie za pomocą OpenIdConnect, kwestie sesji celowo pomijam, ponieważ wykracza ona poza tematykę tego wpisu (podpowiedź -> SignInScheme).
{: style="text-align: justify;"}

```csharp
var options = new OpenIdConnectOptions
            {
                Authority = string.Format("https://login.microsoftonline.com/tfp/{0}/{1}", Configuration["Authentication:AzureAdB2C:TenantName"], Configuration["Authentication:AzureAdB2C:SignInPolicyName"]),
                MetadataAddress = string.Format(Configuration["Authentication:AzureAdB2C:MetadataEndpointUrlTemplate"], Configuration["Authentication:AzureAdB2C:TenantName"], Configuration["Authentication:AzureAdB2C:SignInPolicyName"]),
                ClientId = Configuration["Authentication:AzureAdB2C:ClientId"],
                ClientSecret = Configuration["Authentication:AzureAdB2C:ClientSecret"],
                Events = new OpenIdConnectEvents
                {
                    OnAuthorizationCodeReceived = OnAuthorizationCodeReceived,
                    OnAuthenticationFailed = OnAuthenticationFailed
                },
                ResponseType = OpenIdConnectResponseType.CodeIdToken,
                SignInScheme = CookieAuthenticationDefaults.AuthenticationScheme
            };
            options.Scope.Add($"{Constants.Scopes.NotesServiceAppIdUri}{Constants.Scopes.NotesServiceReadNotesScope}");
            options.Scope.Add($"{Constants.Scopes.NotesServiceAppIdUri}{Constants.Scopes.NotesServiceWriteNotesScope}");
            options.Scope.Add("offline_access");
            app.UseOpenIdConnectAuthentication(options);
```

W powyższym fragmencie kodu na dzień dobry mamy kilka ciekawych operacji. Przede wszystkim w odpowiedzi oczekujemy CodeIdToken, ten enum oznacza zarówno id_token użytkownika, jak również authorization_code, czyli początek naszej drogi z Authorization Code Grant. Dodatkowo z sekcji scope, poza dostępem do naszego API z notatkami, prosimy również o wspomniany w poprzednim wpisie offline\_access. Dzięki temu uzyskamy refresh_token, który przechowywany w token cache pozwoli nam wielokrotnie prosić o access_token do API, bez potrzeby ponownego logowania interaktywnego użytkownika.
{: style="text-align: justify;"}

W przypadku logowania zakończonego powodzeniem dostajemy zdarzenie OnAuthorizationCodeReceived, w którym pobieramy wspomniany refresh_token i nasz pierwszy access_token. Tokeny składujemy w przeportowanym przeze mnie na .NET Core 1.1 token cache (patrz MSALSessionCache.cs).
{: style="text-align: justify;"}

```csharp
private async Task OnAuthorizationCodeReceived(AuthorizationCodeReceivedContext authorizationCodeReceivedContext)
        {
            var code = authorizationCodeReceivedContext.ProtocolMessage.Code;
            string signedInUserID = authorizationCodeReceivedContext.Ticket.Principal.FindFirst(ClaimTypes.NameIdentifier).Value;
            TokenCache userTokenCache = new MSALSessionCache(signedInUserID, authorizationCodeReceivedContext.HttpContext).GetMsalCacheInstance();
            ConfidentialClientApplication cca = new ConfidentialClientApplication(
                Configuration["Authentication:AzureAdB2C:ClientId"],
                string.Format("https://login.microsoftonline.com/tfp/{0}/{1}", Configuration["Authentication:AzureAdB2C:TenantName"], Configuration["Authentication:AzureAdB2C:SignInPolicyName"]),
                Configuration["Authentication:AzureAdB2C:CallbackPath"], 
                new ClientCredential(Configuration["Authentication:AzureAdB2C:ClientSecret"]), 
                userTokenCache, userTokenCache);
            string[] scopes =
            {
               $"{Constants.Scopes.NotesServiceAppIdUri}{Constants.Scopes.NotesServiceReadNotesScope}",
               $"{Constants.Scopes.NotesServiceAppIdUri}{Constants.Scopes.NotesServiceWriteNotesScope}"
            };
            AuthenticationResult result = await cca.AcquireTokenByAuthorizationCodeAsync(code, scopes);
        }
```

W tym miejscu po raz pierwszy spotykamy się z MSAL&#8217;em. Klasa **ConfidentialClientApplication** to nasz prywatny klient, w odróżnieniu od klienta publicznego, reprezentowanego przez **PublicClientApplication**. Na uwagę zasługuje również drugie z obsługiwanych zdarzeń, OnAuthenticationFailed, który gorąco polecam do debuggowania 🙂
{: style="text-align: justify;"}

Jeżeli powyższe fragmenty zakończyły się sukcesem, to w naszym token cache powinniśmy mieć już potrzebne tokeny i możemy przy ich pomocy każdorazowo uzyskiwać access\_token. Kod powyżej używał krótko żyjącego access_code (pierwsza linia), który bezpowrotnie tracimy.
{: style="text-align: justify;"}

W metodach naszego WebAPI, przy każdej z operacji dodawania, czytania lub usuwania notatek, prosimy o dostęp i o odpowiednie Scope używając metody **AcquireTokenSilentAsync** (bez ponownego logowania interaktywnego):

```csharp
private async Task<string> AcquireToken(string[] scopes)
        {
            string signedInUserID = User.Claims.FirstOrDefault(claim =&gt; claim.Type == ClaimTypes.NameIdentifier)?.Value;
            TokenCache userTokenCache = new MSALSessionCache(signedInUserID, this.HttpContext).GetMsalCacheInstance();
            ConfidentialClientApplication cca = new ConfidentialClientApplication(
                Startup.Configuration["Authentication:AzureAdB2C:ClientId"],
                string.Format("https://login.microsoftonline.com/tfp/{0}/{1}", Startup.Configuration["Authentication:AzureAdB2C:TenantName"], Startup.Configuration["Authentication:AzureAdB2C:SignInPolicyName"]),
                Startup.Configuration["Authentication:AzureAdB2C:CallbackPath"],
                new ClientCredential(Startup.Configuration["Authentication:AzureAdB2C:ClientSecret"]),
                userTokenCache, null);

            AuthenticationResult result = await cca.AcquireTokenSilentAsync(scopes, cca.Users.FirstOrDefault());
            return result.AccessToken;
        }
```

Uzyskany access_token dołączamy jako JWT Bearer w nagłówku żądania, które wysyłamy do API, a tam jak już opisałem, token jest sprawdzany i analizowany przez polityki autoryzacyjne.
{: style="text-align: justify;"}

```csharp
[HttpPost]
        public async Task&lt;ActionResult&gt; Delete(int id)
        {
            var token = await AcquireToken(new[] { $"{Constants.Scopes.NotesServiceAppIdUri}{Constants.Scopes.NotesServiceWriteNotesScope}", $"{Constants.Scopes.NotesServiceAppIdUri}{Constants.Scopes.NotesServiceReadNotesScope}" });

            var client = new HttpClient();
            HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Delete, "https://localhost:44397/api/notes/" + id);
            request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", token);

            var response = await client.SendAsync(request);
            return RedirectToAction(nameof(Index));
        }
```

### Podsumowanie

Gorąco zachęcam do ściągnięcia kodu i samodzielnego poeksperymentowania z OAuth. Temat czasami jest nieco skomplikowany, ale przy odrobinie wprawy uzyskasz cenne doświadczenie deweloperskie ze standardem, który obecnie króluje na rynku i jest używany przez większość firm technologicznych. Powyższe przykłady, po wyeliminowaniu polityk B2C mogą być również przydatne podczas tworzenia API/aplikacji na klasyczne Azure AD - pamiętaj jednak, że aplikacje znane z blade'a "Enterprise Applications" i "Application Registrations" obsługują w większości starą wersję protokołu (endpointy v1).
{: style="text-align: justify;"}

Jeżeli masz jakieś uwagi lub pytania, zapraszam do kontaktu!
{: style="text-align: justify;"}