---
title: Jak się uchronić przed blokowaniem OAuth przez Google
date: 2017-04-18T18:28:31+02:00
header:
  teaser: /assets/images/2017/04/2017.04.18-oauth-2-sm-2.png
categories:
  - B2C
tags:
  - B2C
  - Google
  - OAuth
  - WebView
---
Już w najbliższy czwartek (20.04.2017) Google zablokuje "logowanie" przy pomocy protokołu OAuth 2.0 w niektórych aplikacjach mobilnych. Zmiany te wynikają z opublikowanego już w sierpniu 2016 roku wpisu o zwiększeniu przez firmę poziomu bezpieczeństwa i wygody obsługi aplikacji. Wpis można znaleźć na blogu <a href="https://developers.googleblog.com/2016/08/modernizing-oauth-interactions-in-native-apps.html" target="_blank" rel="noopener noreferrer">Google Developers</a>. Poniżej opiszę jak się uchronić przed blokowaniem OAuth przez Google, nie zabraknie również części poświęconej wymaganym zmianom w Azure AD B2C.
{: style="text-align: justify;"}

### Jakie aplikacje narażone są na problemy?

Z opublikowanego komunikatu wynika, że wszystkie aplikacje mobilne, które podczas uwierzytelniania i autoryzacji do kont Google używają natywnej kontrolki WebView (a konkretnie <a title="" href="https://developer.android.com/reference/android/webkit/WebView.html" target="_blank" rel="noopener noreferrer">WebView</a> w Android, <a title="" href="https://developer.apple.com/reference/uikit/uiwebview" target="_blank" rel="noopener noreferrer">UIWebView</a> w iOS, <a href="https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.controls.webview" target="_blank" rel="noopener noreferrer">WebView</a> w Windows Phone itp) nie będą w stanie od czwartku wykonać tego typu operacji. W efekcie użytkownicy wspomnianych aplikacji pozbawieni zostaną możliwości logowania do swojego konta. To z kolei może przełożyć się na sporo innych komplikacji.
{: style="text-align: justify;"}

Podobnie sprawa wygląda w przypadku Azure AD B2C, które przez długi okres (do marca 2017) wspierało tylko taką formę procesu uwierzytelniania i autoryzacji. Praktycznie każda aplikacja B2C/Google+ jest potencjalnie narażona na problemy, jeżeli nie wdrożymy wymaganych poprawek. Na liście są aplikacje stworzone przy pomocy Xamarin, aplikacje używające <a href="https://github.com/AzureAD/microsoft-authentication-library-for-dotnet" target="_blank" rel="noopener noreferrer">MSAL&#8217;a</a>, bibliotek <a href="https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-authentication-libraries" target="_blank" rel="noopener noreferrer">ADAL</a>, <a href="https://github.com/kalemontes/OIDCAndroidLib" target="_blank" rel="noopener noreferrer">OIDCAndroidLib</a> lub <a href="https://github.com/nxtbgthng/OAuth2Client" target="_blank" rel="noopener noreferrer">NXOAuth2Client</a>.
{: style="text-align: justify;"}

### Jakie zmiany musisz wdrożyć, by Twoja aplikacja dalej działała?

Ten fragment przeznaczony jest oczywiście dla wydawców i twórców aplikacji mobilnych. Jako użytkownik musisz cierpliwie poczekać na nową wersję aplikacji, która pewnie będzie musiała przejść długotrwały proces weryfikacji.
{: style="text-align: justify;"}

W klasycznym rozwiązaniu aplikacja, kiedy chce zalogować użytkownika poprzez Google, tworzy kontrolkę WebView, wysyłając go na adres logowania. Po zakończonym logowaniu IdP (Identity Provider) przekierowuje żądanie na ustalony wcześniej adres &#8211; w przypadku B2C i rozwiązań native client jest to zwykle:

```html
urn:ietf:wg:oauth:2.0:oob
```

Aplikacja oczekuje w WebView na określone przekierowanie, odbiera i weryfikuje token z odpowiedzią kończąc w ten sposób proces logowania. W nowym procesie nie możemy użyć WebView i musimy opuścić na chwilę aplikację uruchamiając żądanie w przeglądarce systemowej. Proces przebiega podobnie, ale pojawia się jedna trudność. Skąd przeglądarka systemowa (np Chrome czy Safari) ma wiedzieć, że po otrzymaniu umówionego przekierowania ma oddać sterowanie z powrotem do naszej aplikacji? Możemy w tym miejscu użyć niestandardowych URN, przykładowo:

```html
com.grabarze.marek.myapp://loginredirect
```

 W naszej aplikacji nasłuchujemy (<a href="https://developer.apple.com/library/content/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Inter-AppCommunication/Inter-AppCommunication.html" target="_blank" rel="noopener noreferrer">przykład w iOS</a>) na ten właśnie URN (pamiętaj, że pierwsza część to właśnie nazwa aplikacji) i po otrzymaniu przekierowania dalszy proces przebiega identycznie.
 {: style="text-align: justify;"}

Pozostaje nam już tylko temat B2C i jak skonfigurować usługę, by dla naszej aplikacji dopuszczała niestandardowy URN. We właściwościach aplikacji pojawiła się jakiś czas temu nowa opcja, dzięki której takie ustawienie jest możliwe:

![img](/assets/images/2017/04/2017.04.18-B2C-URN-Settinga.png)

Ostatnia rzecz to zabezpieczenie się przed sytuacją,  kiedy inna aplikacja próbuje podkraść odpowiedź dotyczącą naszej próby logowania z przeglądarki. Możemy się przed takim zdarzeniem uchronić dzięki "pixy" <a href="https://tools.ietf.org/html/rfc7636" target="_blank" rel="noopener noreferrer">Proof Key for Code Exchange</a>. W skrócie generujemy kod, który wysyłany jest wraz z żądaniem, tylko dzięki niemu możemy odszyfrować odpowiedź.
 {: style="text-align: justify;"}

### Kilka słów wyjaśnienia

W powyższym tekście pojawiło się sporo technicznych pojęć, jeżeli cokolwiek sprawiło Ci trudność, umieść proszę swoje pytania w komentarzach, postaram się na nie w miarę możliwości odpowiedzieć.
 {: style="text-align: justify;"}

Nie śledzę na bieżąco ogłoszeń dotyczących OAuth, na szczęście mogę zawsze w tym temacie liczyć na <a href="https://twitter.com/tonyszko" target="_blank" rel="noopener noreferrer">Tomasza Onyszko</a>, polskiego eksperta w dziedzinie bezpieczeństwa i tożsamości. Bez niego pewnie bym się dowiedział o powyższych zmianach w czwartek i miał sporo pracy do zrobienia po godzinach.
 {: style="text-align: justify;"}