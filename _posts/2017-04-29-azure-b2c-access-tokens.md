---
title: Azure B2C Access Tokens. Zabezpiecz swoje API przy pomocy B2C
date: 2017-04-29T13:29:23+02:00
header:
  teaser: /assets/images/2017/03/b2c.png
categories:
  - B2C
tags:
  - API
  - Azure
  - B2C
  - OAuth2.0
  - OpenIdConnect
---
Dzisiejszy wpis jest pierwszą, zdecydowanie bardziej teoretyczną częścią zagadnienia związanego z zabezpieczaniem backendowych API przy pomocy B2C. Skupię się tutaj na authorization code grant flow i funkcjonalności Azure B2C Access Tokens, która pojawiła się jako public preview w końcu marca. Drugi wpis w serii skupi się na zagadnieniach programistycznych, w tym między innymi na tym, jak opisaną funkcjonalność użyć po stronie klienta i po stronie API. Dla utrudnienia i być może w celu zwiększenia poczytności, przykłady będą opierać się o ASP.NET Core 1.1 🙂
{: style="text-align: justify;"}

### Authorization Code Grant Flow

Typowym zastosowaniem dla authorization code grant jest zabezpieczenie dostępu aplikacji klienckich do backendowych API. Jest on częścią OAuth 2.0 , a jego szczegółową specyfikację można znaleźć w <a href="https://tools.ietf.org/html/rfc6749">RFC 6749</a>. Proces ma na celu pozyskanie **access_token** przez aplikację kliencką, który posłuży jej w dalszej kolejności do wywołania zabezpieczonych metod API. W odróżnieniu od klasycznego **id_token** pozyskiwanego w procesie uwierzytelniania, zawiera on dodatkowo **scope** listę zasobów i potrzebnych uprawnień, które aplikacja kliencka żąda/uzyskała w naszym imieniu.
{: style="text-align: justify;"}

Dość istotnym zagadnieniem jest zidentyfikowanie, czy aplikacja kliencka jest publiczna czy prywatna. Do aplikacji publicznych zaliczamy wszelkie aplikacje mobilne lub desktopowe, w których nie jesteśmy w stanie ukryć poświadczeń aplikacji B2C. Aplikacje prywatne, jak strony webowe lub inne API, mogą bezpiecznie przechowywać ClientId i ClientSecret w konfiguracji, bez ryzyka ich upublicznienia (kluczy).
{: style="text-align: justify;"}

Dla aplikacji publicznych wykonywane są następujące kroki:

  1. Aplikacja (reprezentowana przez ClientId) wysyła użytkownika do strony **authorize** Przykładowo <https://login.microsoftonline.com/b2crocks.onmicrosoft.com/oauth2/v2.0/authorize?>
  2. Lista parametrów musi zawierać między innymi **client_id**, **response_type=code** oraz **scope** który żądamy.
  3. Uzyskujemy tym sposobem **authorization_code**, który posłuży nam do uzyskania niezbędnego tokenu.
  4. Aplikacja wysyła POSTa do strony **/token**. Lista parametrów musi zawierać **client_id**, **grant_type=authorization_code**, **scope** oraz uzyskany wcześniej **code**.
  5. Otrzymany tym sposobem **access_token** może być używany w wywołaniach naszego backend API przez dodanie go w nagłówku **Authorization**

W przypadku aplikacji webowych możemy wyjść poza granice authorization code grant flow i "poszaleć" wykonując pełne uwierzytelnianie w oparciu o OpenId Connect, uzyskując claimy użytkownika i **id_token**. W tym miejscu należy się pewnie kilka słów wyjaśnienia. OAuth jest protokołem autoryzacyjnym (tak tak, Auth pochodzi od Authorize, nie od Authenticate). Jest w stanie wykonać jednorazową autoryzację użytkownika/aplikacji. Używając OpenId Connect, które jest uwierzytelniającym dodatkiem do OAuth 2.0, pozyskując id_token, możemy wspierać chociażby SSO - dlatego warto rozważyć jego użycie.  Uzyskiwanie access token przebiega podobnie:
{: style="text-align: justify;"}

  1. Aplikacja wysyła użytkownika do strony **/authorize**.
  2. Lista parametrów musi zawierać między innymi **client_id**, **response_type=code+open_id** oraz **scope** zawierający wartość openid - o tym poniżej.
  3. Uzyskujemy tym sposobem **authorization_code** oraz **id_token**, czyli dostęp do claimów użytkownika.
  4. Wykonujemy walidację **id_token**. Weryfikujemy wystawcę oraz **aud, iat, exp, nonce** upewniając się, że token wystawiony jest dla naszej aplikacji, nie wygasł i nie pochodzi z innego, wcześniejszego żądania. Więcej o walidacji <a href="https://docs.microsoft.com/en-us/azure/active-directory-b2c/active-directory-b2c-reference-tokens#token-validation" target="_blank" rel="noopener noreferrer">tutaj</a>.
  5. Aplikacja wysyła POSTa do strony **/token**. Lista parametrów musi zawierać **client_id**, **grant_type=authorization_code**, **scope** oraz uzyskany wcześniej **code**. Tym razem musimy dorzucić również **client_secret**.  
  6. Otrzymany tym sposobem **access_token** może być używany w wywołaniach naszego backend API przez dodanie go w nagłówku **Authorization**.

Zanim przejdziemy do Azure B2C Access Tokens i konfiguracji scope, chciałbym jeszcze wspomnieć o standardowych scope, o które możemy prosić (gdy jest ich więcej, oddzielamy je spacjami):
{: style="text-align: justify;"}

- **offline_access**, który umożliwia uzyskanie refresh tokena.
- **openid** uprawniający do uzyskania  **id_token**.

### B2C Access Tokens &#8211; Konfiguracja własnych scope

Do momentu ogłoszenia <a href="https://azure.microsoft.com/en-us/blog/azure-ad-b2c-access-tokens-now-in-public-preview/" target="_blank" rel="noopener noreferrer">public preview</a>, poza dwoma standardowymi scope, mogliśmy użyć jedynie clientId. Oznaczało to, że zarówno aplikacja kliencka, jak też backend, musiały współdzielić ten sam Id i w ten sposób autoryzować dostęp "samej sobie"do API. To z kolei, przy więcej niż jednej aplikacji klienckiej oznaczało poważne utrudnienia (które na szczęście dawało się częściowo obejść).
{: style="text-align: justify;"}

W chwili obecnej możemy tworzyć własne scope w aplikacji backendowej, a następnie uprawniać poszczególne aplikacje klienckie do ich używania. Myślę, że będzie łatwiej, gdy opiszę to na przykładzie i posłużę się grafikami z bloga MSDN.
{: style="text-align: justify;"}

[Przykład]. Chcielibyśmy umożliwić naszym klientom z B2C tworzenie i czytanie notatek pod produktami, które sprzedajemy. Tworzymy API, które taką funkcjonalność wystawia i definiujemy dwa scope.
{: style="text-align: justify;"}

![img](/assets/images/2017/04/0fb084eb-4770-4c53-8984-2f981999ddd6.png)

Mamy również dwie aplikacje. Pierwsza aplikacja wyświetla produkty i pokazuje notatki użytkowników pod produktami. Druga aplikacja pozwala wykonywać zakupy i dodawać notatki do zakupionych produktów. Dość często zdarza się, że używając OAuth 2.0 i autoryzując aplikację, jesteśmy po zalogowaniu pytani o wyrażenie zgody na np: "Allow this app to post notes under your name", "llow this app to read your notes". To nic innego jak scope, a konkretnie "user consent" zezwalający aplikacji na wykonywanie pewnych operacji na backendowych API w Twoim imieniu. Przytoczony wcześniej "openid" scope, to z kolei zezwolenie na dostęp do Twoich claimów, czyli maila, nazwiska, adresu itp.
{: style="text-align: justify;"}

W B2C tego typu pytanie się nie pojawia, to my jako administratorzy mamy kontrolę nad tym, która aplikacja ma uprawnienia do naszych API i jakie są to uprawnienia. Dla naszych aplikacji klienckich konfiguracja wygląda następująco:
{: style="text-align: justify;"}

![img](/assets/images/2017/04/e607c1dd-1a5d-4349-98a4-80db5172e293.png)

W tym momencie mamy pełną kontrolę, że aplikacje klienckie pomimo zalogowania użytkownika mają dostęp do jedynie wskazanych przez nas API lub nawet poszczególnych ich metod.
{: style="text-align: justify;"}

### Następne kroki

Myślę, że po zapoznaniu się z teorią, należy przejść do praktyki. W następnym wpisie zaprezentuję jak w oparciu o ASP.NET Core 1.1 możemy stworzyć API i jak wystawić/autoryzować scope na poszczególnych kontrolerach lub ich metodach. Nie zabraknie również zabawy z claimami (dopuszczanie np tylko tych użytkowników, którzy zalogowali się przez Facebook, albo użytkowników którzy po raz pierwszy nas odwiedzają).
{: style="text-align: justify;"}

Dopełnieniem części praktycznej będzie przykładowy kod aplikacji klienckiej, która przy użyciu  authorization code grant flow i uzyskanego access_token wysyła żądania do naszego API.
{: style="text-align: justify;"}