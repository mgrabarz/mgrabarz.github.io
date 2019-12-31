---
title: Seria wpisów o Azure API Management
date: 2017-09-30T19:20:11+02:00
header:
  teaser: /assets/images/2017/09/12687690_9b8e70564a_z.jpg
categories:
  - API Management
tags:
  - API
  - API Management
  - APIM
  - Azure
- - Governance
---
Muszę przyznać, że trochę się ostatnio ociągałem w temacie blogowania. W celu poprawienia aktywności postanowiłem stworzyć serię wpisów na konkretny temat. Same wpisy będą zdecydowanie krótsze, poruszające tylko wybrany aspekt opisywanej technologii. Na pierwszy ogień idzie Azure API Management, którym miałem okazję się sporo zajmować w ostatnich miesiącach.

## Czym jest Azure API Management?

Jest to usługa Azure która pozwala stworzyć portal dla programistów, na którym publikowane są wszystkie korporacyjne API tworzone przez inne grupy programistów.  Na pierwszy rzut oka bardzo niewiele, ale kiedy przyjrzymy się jakie dodatkowe funkcjonalności portal zapewnia. Nagle okazuje się że mamy pełną kontrolę nad API w całej firmie, w tym:

  * Kto i jak używa usług
  * Statystyki użycia
  * Stworzenie potężnego reverse proxy dla wszystkich usług, w tym tych z on-premises
  * Zarządzanie security naszych API
  * No i przede wszystkich katalog usług, z opisami, przykładami i możliwością próbowania z poziomu portalu

W kolejnych postach postaram się przybliżyć Ci możliwości Azure API Management i pokażę jak całą usługę przeklikać od początku do końca. Wspólnie zrealizujemy wszystkie typowe scenariusze.

## Pierwsze kroki i pierwsze&#8230; porady

Jeżeli dopiero teraz rozważasz na poważnie rozpoczęcie przygody z Azure API Management, a Twoje zasoby w Azure tworzone były już od dłuższego czasu, możesz napotkać na jeden poważny problem.  Szczególnie jeśli nie miałeś jasnej polityki co do preferowanej lokalizacji dla twoich zasobów, może się nagle okazać że maszyny wirtualne masz w North Europe (bo taniej), niektóre WebApps w West Europe (bo bliżej), a jeszcze inne zasoby w trzecim regionie.

Gdzie w takim razie postawić instancję naszego APIM? Będziesz miał pewnie ciężki wybór, zależny od tego gdzie są API, jakie to usługi, przez kogo są wołane. Warto ten problem rozważyć zanim stworzysz i skonfigurujesz APIM. Druga sprawa to polityka gdzie trzymać same API, wypadałoby ją stworzyć o wiele wcześniej.  Co jeszcze możemy zrobić:

  * APIM jest w stanie działać w wielu regionach, ale  wtedy cena wynosi niebagatelne 2849$/miesiąc/region.
  * Jeśli stworzyłeś już instację APIM i chcesz ją przenieść, masz do dyspozycji wiele <a href="https://docs.microsoft.com/en-us/azure/api-management/api-management-faq#how-do-i-copy-my-api-management-service-instance-to-a-new-instance" target="_blank" rel="noopener">narzędzi </a>oraz polecenia Import i Export w Powershell dla poszczególnych API.

## Następne kroki?

W kolejnym wpisie przedstawię model security w API Management oraz krótką radę, jak przy pomocy Powershell (bo w portalu się nie da) zmienić kilka ustawień ułatwiających dalsze zarządzanie.

* * *

Photo credit: [phooky](https://www.flickr.com/photos/phooky/12687690/) via [VisualHunt](https://visualhunt.com/re/296da9) /  [CC BY-SA](http://creativecommons.org/licenses/by-sa/2.0/)