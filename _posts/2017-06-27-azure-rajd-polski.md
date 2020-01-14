---
title: Jak wykorzystywany jest Azure w Rajdzie Polski
date: 2017-06-27T00:34:39+02:00
header:
  teaser: /assets/images/2017/06/rally-single-seater-racing-car-machine-sardinia-1-1.jpg
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
{: style="text-align: justify;"}

W tym roku Rajd Polski organizowany jest po raz 74 i podobnie jak przed rokiem jest etapem WRC (Rajdowych Mistrzostw Świata). Historycznie jest to drugi po Monte Carlo najstarszy rajd na świecie, przy czym kilka edycji się nie odbyło z powodu wielkiego kryzysu i II wojny światowej. Jest to wydarzenie, na które corocznie przyjeżdżają tysięce kibiców z całego świata i śledzą zmagania najlepszych kierowców rajdowych.
{: style="text-align: justify;"}

### Opis wymagań systemu

W przedstawionych wymaganiach skupię się na opisie rozwiązania dla wielu klientów. Oto podstawowe założenia:
{: style="text-align: justify;"}

- Pierwszym, kluczowym elementem będzie możliwość definiowania klientów, a dla nich wydarzeń.
- Klienci po zalogowaniu się do serwisu i stworzeniu wydarzenia mają możliwość konfiguracji stref, które to strefy mogą się wzajemnie zagnieżdżać. Przez strefę rozumiem tutaj różne obszary, w których uczestnicy wydarzenia będą się mogli poruszać. Mogą to być miejsca stojące, miejsca w pierwszym rzędzie, obszary dostępne jedynie dla obsługi, trybuny itp.
- Każda ze stref może mieć wiele bramek wejściowych, w szczególności przy dużych wydarzeniach może to być 10-20 jednoczesnych bramek  (z różnych krańców strefy). Bramki **muszą się wzajemnie informować** o tym kto wszedł i wyszedł, aby nie było możliwości podania biletu "przez płot", albo wielokrotnego użycia biletów jednorazowych.
- System pozwala zdefiniować wiele typów biletów, które działają w różnych strefach (relacja n-m). Na przykład bilet na trybuny pozwala przejść przez jedną strefę, aby dostać się do innej, zagnieżdżonej.
- Klient ma wreszcie możliwość wgrania biletów, które następnie będą weryfikowane przez aplikację natywną przy bramkach (desktop, mobile, tablet) z użyciem skanowania kodów kreskowych.

### Zarys architektury

Do dyspozycji mamy oczywiście wszelkie komponenty Azure, ale z uwagi że będzie to usługa dostępna dla wielu potencjalnych klientów, postaram się przedstawić skalowalne i tanie rozwiązanie.
{: style="text-align: justify;"}

Centralnym zagadnieniem, które musimy rozwiązać jest zapewnienie komunikacji pomiędzy poszczególnymi bramkami. W moim rozwiązaniu używany jest Azure Service Bus, a konkretnie kolejki <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions" target="_blank" rel="noopener">Topic/Subscription</a>. Model ten pozwala poszczególnym bramkom zgłosić fakt wejścia osoby (na wspólny topic). Pozostałe bramki będące subskrybentami zostaną o tym fakcie poinformowane. Nasuwa się też kilka dodatkowych przemyśleń:
{: style="text-align: justify;"}

- W przypadku problemów z połączeniem z Internetem aplikacja może tymczasowo przechowywać zgromadzone komunikaty, które zostaną wysłane na topic.
- W przypadku powyższym zgłoszenia czekają w chmurze na dedykowanej dla bramki subskrypcji, można je ona pobrać gdy wróci połączenie.
- W przypadku wyjątków komunikaty możemy ponawiać lub transakcyjnie odbierać.
- Możemy "na zapas" zdefiniować wiele bramek. Bramki podłączone awaryjne do aplikacji (gdy na przykład przepustowość jest niewystarczająca i dostawiamy bramki) mogą dostać całą historię wejść i ją zaaplikować w parę chwil.
- W celu optymalizacji ruchu w kolejkach i w sieci używamy <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-messaging-sql-filter" target="_blank" rel="noopener">SqlFilter</a>, który sprawia, że komunikaty w subskrypcjach trafiają tylko do bramek będących w tym samym regionie. To ważne usprawnienie, które pozwoliło mi zmniejszyć ruch niemal o rząd wielkości.
- Przy użyciu powyższej klasy możemy wreszcie opanować nasz model wielu klientów. Dla klientów premium możemy użyć kolejek w trybie Premium, z dedykowanym Message Unit(s). Dla wersji dla mniejszych klientów, współdzielonych kolejek z filtrem.

![img](/assets/images/2017/06/TopicSubscription.png)

Kiedy już opanujemy zagadnienia komunikacyjne, musimy zatroszczyć się o tematykę bezpieczeństwa. Definiujemy kto i jak może wysyłać komunikaty na topic i odbierać dane z subskrypcji. W tym celu możemy wygenerować unikalny <a href="https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-sas" target="_blank" rel="noopener">SAS</a> (shared access signature) dla instancji naszej aplikacji. Takie podejście nie zabezpieczy nas niestety przed tym, że na topic nie trafią spreparowane wiadomości. W tym miejscu warto przyjrzeć się <a href="https://azure.microsoft.com/en-us/services/functions/" target="_blank" rel="noopener">Azure Functions</a>, które może zweryfikować wystawiony dla instancji **token** i w jej imieniu wysłać zaufaną wiadomość do topica. Wyzwalaczem dla funkcji będzie oczywiście żądanie POST przez HTTPS.
{: style="text-align: justify;"}

### Zarządzenie konfiguracją i przechowywanie danych historycznych

Jak już wcześniej zasygnalizowałem, aplikacja wymaga części frontendowej. W tym celu tworzymy prostą aplikację działającą na <a href="https://azure.microsoft.com/en-us/services/app-service/web/" target="_blank" rel="noopener">AppService</a>. Dodatkowo musimy zaimlementować API, które posłuży nam do pobierania konfiguracji, tutaj znów możemy posłużyć się Azure Functions.
{: style="text-align: justify;"}

Dane konfiguracji mogą być przechowywane np w bazie <a href="https://azure.microsoft.com/en-us/services/sql-database/" target="_blank" rel="noopener">SQL</a>, ja natomiast preferuję <a href="https://azure.microsoft.com/en-us/services/storage/tables/" target="_blank" rel="noopener">Azure Table Storage</a>. Podejście takie zapewnia nam bardzo niski koszt rozwiązania, jak również dobre skalowanie. Możemy tworzyć wiele kont magazynu i balansować obciążenie w zależności od wymagań klientów. Z kolei dzięki Azure Functions możemy uniknąć bezpośredniego dostępu aplikacji do danych konfiguracyjnych. Myślę, że na podstawie samych wymagań z pierwszego paragrafu da się również dość łatwo zdefiniować model danych.
{: style="text-align: justify;"}

Sprytnym sposobem na opanowanie tematu logowania jest stworzenie dodatkowej subskrypcji w kolejce, która bez filtra loguje wszystkie wejścia (znów do Table Storage). Oczywiście komunikat wyzwala kolejną Azure Functions (logger). Można się łatwo domyślić, że to jeden z moich ulubionych komponentów Azure.
{: style="text-align: justify;"}

Zebrane dane mogą być następnie przedstawiane w postaci przystępnych wykresów w PowerBI. W moim przypadku są to raporty online, które pozwalają na żywo obserwować ruch na bramkach. Szczególnie przydatne dla organizatorów są raporty obrazujące ilość osób w strefach (względy bezpieczeństwa). Raporty pozwalają również analizować pełne statystki po wydarzeniu.
{: style="text-align: justify;"}

Całość rozwiązania dopełnia Application Insights (logowanie aplikacyjne z kodu) oraz katalog użytkowników (np Azure AD). Dodatkowo najnudniejsza część, czyli aplikacja na bramkach, która skanuje kody i wysyła/odbiera komunikaty dotyczące wejść.
{: style="text-align: justify;"}

### Podsumowanie

Całość opisanej architektury przedstawiona jest na poniższym diagramie:

![img](/assets/images/2017/06/EventArchitecture.jpg)

Użyte komponenty dają się łatwo skalować, a przy dobrze zdefiniowanym &#8220;Infrastructure as Code&#8221; możemy nasze środowisko szybko tworzyć i likwidować (pozostawiając dane). Do tego typu zadań idealnie nadają się <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates" target="_blank" rel="noopener">szablony ARM</a>, których przykłady można znaleźć na <a href="https://github.com/Azure/azure-quickstart-templates" target="_blank" rel="noopener">Azure Quickstart Templates</a>.
{: style="text-align: justify;"}

Koszty rozwiązania też są satysfakcjonujące. Większość komponentów rozliczana jest w modelu czasowym, albo jak w Azure Functions, za poszczególne wywołania. Uruchomienie rozwiązania na potrzeby czterodniowego rajdu nie kosztuje więcej niż solidny posiłek w Warszawie, głównie przez opłaty ServiceBus. Mając na uwadze obsługę tysięcy osób i miliona komunikatów dziennie, cena wydaje się być mimo wszystko całkiem niezła.
{: style="text-align: justify;"}

_Nie pozostaje mi nic innego jak oficjalnie zaprosić Cię na Rajd Polski. Impreza dostarcza wielu emocji, a wakacyjno-rajdowa atmosfera Mikołajek jest naprawdę świetna. Daj proszę znać, czy podobał Ci się mój wpis, lub jeśli masz ochotę porozmawiać przy kawie w Mikołajkach w najbliższy weekend!_
{: style="text-align: justify;"}

Photo via [Visualhunt.com](https://visualhunt.com/re/994773)
{: style="text-align: justify;"}