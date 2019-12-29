---
id: 468
title: Jak wykorzystywany jest Azure w Rajdzie Polski
date: 2017-06-27T00:34:39+02:00
author: Grabarz
layout: post
guid: https://marekgrabarz.pl/?p=468
permalink: /2017/06/azure-rajd-polski/
image: /wp-content/uploads/2017/06/rally-single-seater-racing-car-machine-sardinia-1-1.jpg
categories:
  - Architecture
tags:
  - Architecture
  - Azure
  - Functions
  - Rally
  - ServiceBus
  - WRC
---
Inspiracją do napisania dzisiejszego wpisu jest zaczynający się w najbliższy czwartek <a href="http://www.rajdpolski.pl/" target="_blank" rel="noopener">Rajd Polski</a>. Na jednej z podstron bloga wspominałem, że stworzyłem kawałek chmurowego oprogramowania, które w tego typu wydarzeniach jest wykorzystywane. Poniżej przedstawię wymagania dotyczące systemu oraz kilka przemyśleń jak rozwiązać zagadnienie przy pomocy komponentów dostępnych w Azure. Zanim jednak przejdziemy do szczegółów i architektury napiszę kilka słów o samym rajdzie.

W tym roku Rajd Polski organizowany jest po raz 74 i podobnie jak przed rokiem jest etapem WRC (Rajdowych Mistrzostw Świata). Historycznie jest to drugi po Monte Carlo najstarszy rajd na świecie, przy czym kilka edycji się nie odbyło z powodu wielkiego kryzysu i II wojny światowej. Jest to wydarzenie, na które corocznie przyjeżdżają tysięce kibiców z całego świata i śledzą zmagania najlepszych kierowców rajdowych.

### Opis wymagań systemu

W przedstawionych wymaganiach skupię się na opisie rozwiązania dla wielu klientów. Oto podstawowe założenia:

  * Pierwszym, kluczowym elementem będzie możliwość definiowania klientów, a dla nich wydarzeń.
  * Klienci po zalogowaniu się do serwisu i stworzeniu wydarzenia mają możliwość konfiguracji stref, które to strefy mogą się wzajemnie zagnieżdżać. Przez strefę rozumiem tutaj różne obszary, w których uczestnicy wydarzenia będą się mogli poruszać. Mogą to być miejsca stojące, miejsca w pierwszym rzędzie, obszary dostępne jedynie dla obsługi, trybuny itp.
  * Każda ze stref może mieć wiele bramek wejściowych, w szczególności przy dużych wydarzeniach może to być 10-20 jednoczesnych bramek  (z różnych krańców strefy). Bramki **muszą się wzajemnie informować** o tym kto wszedł i wyszedł, aby nie było możliwości podania biletu &#8220;przez płot&#8221;, albo wielokrotnego użycia biletów jednorazowych.
  * System pozwala zdefiniować wiele typów biletów, które działają w różnych strefach (relacja n-m). Na przykład bilet na trybuny pozwala przejść przez jedną strefę, aby dostać się do innej, zagnieżdżonej.
  * Klient ma wreszcie możliwość wgrania biletów, które następnie będą weryfikowane przez aplikację natywną przy bramkach (desktop, mobile, tablet) z użyciem skanowania kodów kreskowych.

### Zarys architektury

Do dyspozycji mamy oczywiście wszelkie komponenty Azure, ale z uwagi że będzie to usługa dostępna dla wielu potencjalnych klientów, postaram się przedstawić skalowalne i tanie rozwiązanie.

Centralnym zagadnieniem, które musimy rozwiązać jest zapewnienie komunikacji pomiędzy poszczególnymi bramkami. W moim rozwiązaniu używany jest Azure Service Bus, a konkretnie kolejki <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions" target="_blank" rel="noopener">Topic/Subscription</a>. Model ten pozwala poszczególnym bramkom zgłosić fakt wejścia osoby (na wspólny topic). Pozostałe bramki będące subskrybentami zostaną o tym fakcie poinformowane. Nasuwa się też kilka dodatkowych przemyśleń:

  * W przypadku problemów z połączeniem z Internetem aplikacja może tymczasowo przechowywać zgromadzone komunikaty, które zostaną wysłane na topic.
  * W przypadku powyższym zgłoszenia czekają w chmurze na dedykowanej dla bramki subskrypcji, można je ona pobrać gdy wróci połączenie.
  * W przypadku wyjątków komunikaty możemy ponawiać lub transakcyjnie odbierać.
  * Możemy &#8220;na zapas&#8221; zdefiniować wiele bramek. Bramki podłączone awaryjne do aplikacji (gdy na przykład przepustowość jest niewystarczająca i dostawiamy bramki) mogą dostać całą historię wejść i ją zaaplikować w parę chwil.
  * W celu optymalizacji ruchu w kolejkach i w sieci używamy <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-messaging-sql-filter" target="_blank" rel="noopener">SqlFilter</a>, który sprawia, że komunikaty w subskrypcjach trafiają tylko do bramek będących w tym samym regionie. To ważne usprawnienie, które pozwoliło mi zmniejszyć ruch niemal o rząd wielkości.
  * Przy użyciu powyższej klasy możemy wreszcie opanować nasz model wielu klientów. Dla klientów premium możemy użyć kolejek w trybie Premium, z dedykowanym Message Unit(s). Dla wersji dla mniejszych klientów, współdzielonych kolejek z filtrem.<figure id="attachment_478" aria-describedby="caption-attachment-478" style="width: 521px" class="wp-caption aligncenter">

<img class="wp-image-478 size-full" src="https://marekgrabarz.pl/wp-content/uploads/2017/06/TopicSubscription.png" alt="topic-subscription" width="521" height="462" srcset="https://marekgrabarz.pl/wp-content/uploads/2017/06/TopicSubscription.png 521w, https://marekgrabarz.pl/wp-content/uploads/2017/06/TopicSubscription-300x266.png 300w" sizes="(max-width: 521px) 100vw, 521px" /> <figcaption id="caption-attachment-478" class="wp-caption-text">Model topic-subscription</figcaption></figure> 

Kiedy już opanujemy zagadnienia komunikacyjne, musimy zatroszczyć się o tematykę bezpieczeństwa. Definiujemy kto i jak może wysyłać komunikaty na topic i odbierać dane z subskrypcji. W tym celu możemy wygenerować unikalny <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-sas" target="_blank" rel="noopener">SAS</a> (shared access signature) dla instancji naszej aplikacji. Takie podejście nie zabezpieczy nas niestety przed tym, że na topic nie trafią spreparowane wiadomości. W tym miejscu warto przyjrzeć się <a href="https://azure.microsoft.com/en-us/services/functions/" target="_blank" rel="noopener">Azure Functions</a>, które może zweryfikować wystawiony dla instancji &#8220;token&#8221; i w jej imieniu wysłać zaufaną wiadomość do topic&#8217;a. Wyzwalaczem dla funkcji będzie oczywiście żądanie POST przez HTTPS.

### Zarządzenie konfiguracją i przechowywanie danych historycznych

Jak już wcześniej zasygnalizowałem, aplikacja wymaga części frontendowej. W tym celu tworzymy prostą aplikację działającą na <a href="https://azure.microsoft.com/en-us/services/app-service/web/" target="_blank" rel="noopener">AppService</a>. Dodatkowo musimy zaimlementować API, które posłuży nam do pobierania konfiguracji, tutaj znów możemy posłużyć się Azure Functions.

Dane konfiguracji mogą być przechowywane np w bazie <a href="https://azure.microsoft.com/en-us/services/sql-database/" target="_blank" rel="noopener">SQL</a>, ja natomiast preferuję <a href="https://azure.microsoft.com/en-us/services/storage/tables/" target="_blank" rel="noopener">Azure Table Storage</a>. Podejście takie zapewnia nam bardzo niski koszt rozwiązania, jak również dobre skalowanie. Możemy tworzyć wiele kont magazynu i balansować obciążenie w zależności od wymagań klientów. Z kolei dzięki Azure Functions możemy uniknąć bezpośredniego dostępu aplikacji do danych konfiguracyjnych. Myślę, że na podstawie samych wymagań z pierwszego paragrafu da się również dość łatwo zdefiniować model danych.

Sprytnym sposobem na opanowanie tematu logowania jest stworzenie dodatkowej subskrypcji w kolejce, która bez filtra loguje wszystkie wejścia (znów do Table Storage). Oczywiście komunikat wyzwala kolejną Azure Functions (logger). Można się łatwo domyślić, że to jeden z moich ulubionych komponentów Azure.

Zebrane dane mogą być następnie przedstawiane w postaci przystępnych wykresów w PowerBI. W moim przypadku są to raporty online, które pozwalają na żywo obserwować ruch na bramkach. Szczególnie przydatne dla organizatorów są raporty obrazujące ilość osób w strefach (względy bezpieczeństwa). Raporty pozwalają również analizować pełne statystki po wydarzeniu.

Całość rozwiązania dopełnia Application Insights (logowanie aplikacyjne z kodu) oraz katalog użytkowników (np Azure AD). Dodatkowo najnudniejsza część, czyli aplikacja na bramkach, która skanuje kody i wysyła/odbiera komunikaty dotyczące wejść.

### Podsumowanie

Całość opisanej architektury przedstawiona jest na poniższym diagramie:

<img class="aligncenter wp-image-479 size-full" src="https://marekgrabarz.pl/wp-content/uploads/2017/06/EventArchitecture.jpg" alt="system architecture" width="699" height="771" srcset="https://marekgrabarz.pl/wp-content/uploads/2017/06/EventArchitecture.jpg 699w, https://marekgrabarz.pl/wp-content/uploads/2017/06/EventArchitecture-272x300.jpg 272w" sizes="(max-width: 699px) 100vw, 699px" /> 

Użyte komponenty dają się łatwo skalować, a przy dobrze zdefiniowanym &#8220;Infrastructure as Code&#8221; możemy nasze środowisko szybko tworzyć i likwidować (pozostawiając dane). Do tego typu zadań idealnie nadają się <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates" target="_blank" rel="noopener">szablony ARM</a>, których przykłady można znaleźć na <a href="https://github.com/Azure/azure-quickstart-templates" target="_blank" rel="noopener">Azure Quickstart Templates</a>.

Koszty rozwiązania też są satysfakcjonujące. Większość komponentów rozliczana jest w modelu czasowym, albo jak w Azure Functions, za poszczególne wywołania. Uruchomienie rozwiązania na potrzeby czterodniowego rajdu nie kosztuje więcej niż solidny posiłek w Warszawie, głównie przez opłaty ServiceBus. Mając na uwadze obsługę tysięcy osób i miliona komunikatów dziennie, cena wydaje się być mimo wszystko całkiem niezła.

_Nie pozostaje mi nic innego jak oficjalnie zaprosić Cię na Rajd Polski. Impreza dostarcza wielu emocji, a wakacyjno-rajdowa atmosfera Mikołajek jest naprawdę świetna. Daj proszę znać, czy podobał Ci się mój wpis, lub jeśli masz ochotę porozmawiać przy kawie w Mikołajkach w najbliższy weekend!_

&#8212;

Photo via [Visualhunt.com](https://visualhunt.com/re/994773)