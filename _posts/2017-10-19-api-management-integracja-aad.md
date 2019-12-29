---
id: 551
title: 'API Management &#8211; Integracja z AAD'
date: 2017-10-19T23:36:53+02:00
author: Grabarz
layout: post
guid: https://marekgrabarz.pl/?p=551
permalink: /2017/10/api-management-integracja-aad/
image: /wp-content/uploads/2017/10/33715694656_7edc394988_c.jpg
categories:
  - AAD
  - API Management
tags:
  - AAD
  - API Management
  - APIM
  - Azure
  - B2C
---
W [poprzednim](https://marekgrabarz.pl/2017/10/api-management-security/) artykule wspomniałem, że następny wpis poświęcę opisowi integracji API Management  z Azure Active Directory. Dzisiaj postaram się szybko opisać proces konfiguracji i pokazać Ci kilka ciekawych opcji związanych z zarządzaniem tożsamością w usłudze.

### Logowanie do Developer Portalu przez AAD

Na wstępie warto zaznaczyć, że domyślną formą logowania dla deweloperów do API Management jest local account. W skrócie każdy z deweloperów ma możliwość założenia konta w oparciu o konto email. W zależności od ustawień możemy wszystkich anonimowych (niezalogowanych) użytkowników przekierować na stronę logowania, zmuszając ich tym sposobem do rejestracji.

Jeżeli Wasz API Management będzie głównie używany wewnętrznie w organizacji, a nie przez osoby z zewnątrz, dość szybko zgłoszą się do Was smutni panowie z InfoSec z prośbą, by dostęp do portalu możliwy był tylko w oparciu o firmowe konta. Nie ukrywam, że jestem zdecydowanym zwolennikiem takiego podejścia, które ma sporą ilość zalet:

  * Hasło podlega politykom zarządzanym przez administratorów
  * Mamy możliwość podpięcia MFA
  * Mamy możliwość korzystania z SSO
  * Kiedy opuścimy naszą firmę automatycznie tracimy dostęp do usługi wraz z wyłączeniem konta organizacyjnego

W związku z faktem, że poruszamy się w świecie Microsoft Azure, dość oczywistym wyborem jest właśnie integracja z Azure Active Directory. Niestety opcja ta jest dostępna jedynie w wersji <a href="https://azure.microsoft.com/en-us/pricing/details/api-management/" target="_blank" rel="noopener">Developer oraz Premium</a> naszego API Management. Sporo osób narzeka na to ograniczenie, ale są szanse na zmiany w przyszłości (np dodanie nowego wariantu usługi).

Proces integracji jest dość dobrze opisany <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-howto-aad" target="_blank" rel="noopener">tutaj</a>, na uwagę zasługują dwie rzeczy:

  * Czytając dokumentację można odnieść wrażenie, że aplikację w AAD należy stworzyć w starym portalu. Oczywiście nowy portal jest w zupełności wystarczający. <a href="https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-integrating-applications" target="_blank" rel="noopener">Ten </a>wpis pewnie Ci trochę w tym pomoże.
  * Przy tworzeniu aplikacji AAD musimy nadać jej dwa scope, <a href="https://msdn.microsoft.com/en-us/library/azure/ad/graph/howto/azure-ad-graph-api-permission-scopes" target="_blank" rel="noopener">User.Read oraz Directory.Read.Al</a>l. Drugi stanowi pewien problem, ponieważ jest uprawnieniem bezpośrednim i wymaga zgody administratora naszego AAD. Musimy go przekonać, by zezwolił API Management na buszowanie po liście użytkowników i grup.  
<img class="alignnone wp-image-554 size-full" src="https://marekgrabarz.pl/wp-content/uploads/2017/10/grantpermissions.png" alt="" width="727" height="263" srcset="https://marekgrabarz.pl/wp-content/uploads/2017/10/grantpermissions.png 727w, https://marekgrabarz.pl/wp-content/uploads/2017/10/grantpermissions-300x109.png 300w" sizes="(max-width: 727px) 100vw, 727px" /> 

### Zarządzanie widocznością API przy pomocy grup AAD

Jeżeli udało Ci się zakończyć powyższe kroki członkowie domeny będą mogli się rejestrować i logować do API Management przy pomocy swoich kont służbowych. Nie zapomnij wyłączyć rejestracji przez local account!

W związku z tym, że APIM może pobierać informacje o grupach w AAD (Directory.Read.All), dostajemy nową funkcję.  Wreszcie możemy dodawać nowe role (poza trzema wbudowanymi, opisanymi w moim poprzednim artykule). Tym sposobem część API lub produktów jesteśmy w stanie opublikować dla wybranych grup użytkowników.  Z kolei członkostwo w grupach zarządzane jest z poziomu AAD, lub nawet samego on-premises AD, jeśli zdecydowaliśmy się wcześniej na takie podejście. Oczywiście członkiem grupy może być inna grupa, co daje nam całkiem dużą elastyczność.

### Inne źródła tożsamości w API Management

Poza local account i AAD usługa pozwala również skonfigurować dostęp przy pomocy kont społecznościowych (Facebook, Twitter, Google+, Microsoft Account). Na szczególną uwagę zasługuje jednak moja ulubiona usługa, szwajcarski scyzoryk jeśli chodzi o tożsamość &#8211; Azure AD B2C.

Użycie B2C nie oznacza jedynie, że możemy dać dostęp do APIM naszym klientom zewnętrznym. Przy pomocy <a href="https://docs.microsoft.com/en-us/azure/active-directory-b2c/active-directory-b2c-reference-trustframeworks-defined-ief-custom" target="_blank" rel="noopener">Identity Experience Framework</a> i custom policies, mamy praktycznie nieskończone możliwości:

  * Możemy zintegrować w B2C kilka tenantów AAD
  * Podpiąć się pod firmowego ADFSa
  * Skorzystać z dowolnego innego źródła tożsamości opartego o SAML 2.0 lub WS-Fed (ten drugi nie jest oficjalnie wspierany, ale u mnie działa).
  * Zebrać przy rejestracji dodatkowe informacje
  * Umożliwić rejestrację tylko na mailowe zaproszenie (link aktywacyjny)
  * I wreszcie, skombinować to co powyżej i wiele, wiele więcej w rozwiązanie spełniające wszystkie nasze potrzeby

Tak przy okazji, IEF w B2C nie należy do rzeczy najprostszych. Mam kilka pomysłów na wpisy na blogu w tym temacie, ale zanim to zrobię, chciałbym się dowiedzieć, czy ktokolwiek jest tym tematem zainteresowany! Będę wdzięczny za wszelkie komentarze 🙂

Tymczasem już dziś zapraszam Cię do następnego wpisu o APIM. Wspólnie dodamy pierwsze API i postaramy się je poprawnie zabezpieczyć.

&#8212;

Photo credit: [Got Credit](https://www.flickr.com/photos/gotcredit/33715694656/) via [Visualhunt.com](https://visualhunt.com/re/aeb13d) /  [CC BY](http://creativecommons.org/licenses/by/2.0/)