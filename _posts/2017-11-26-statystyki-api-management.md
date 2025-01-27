---
title: Statystyki w API Management
date: 2017-11-26T22:53:41+01:00
header:
  teaser: /assets/images/2017/11/plane-schedule-board.jpg
categories:
  - APIM
tags:
  - API Management
  - APIM
  - EventHub
  - Functions
  - LogicApp
  - Machine Learning
  - PowerBI
  - Stream Analytics
  - Governance
---

Kilka tygodni temu, w ramach przeglądania newsów ze świata Azure, zauważyłem ciekawe rozwiązanie. Grupa produktowa z API Management stworzyła <a href="http://aka.ms/apimpbi" target="_blank" rel="noopener">szablon</a>, który integruje tę usługę z PowerBI. Stworzone raporty fajnie pokazują podstawowe statystyki API Management z rozbiciem na poszczególne API, metody, produkty i ich subskrybentów. Dodatkowo smaczku dodaje Machine Learning wykrywający niepoprawne użycie i anomalie we wzorcach wywołań.
{: style="text-align: justify;"}

Jakiś czas temu miałem okazję wdrożyć go u siebie i solidnie przetestować. Dzisiaj postaram się Wam przedstawić samo rozwiązanie i kilka spostrzeżeń z użycia.
{: style="text-align: justify;"}

### Jakie statystyki API Management prezentowane są w PowerBI?

Wdrażamy raport przez kliknięcie &#8220;Get it now&#8221;, przechodzimy przez prosty kreator konfiguracji (czasem wolę angielskie nazwy - tutaj setup wizard) i po paru minutach rozwiązanie ląduje w naszej subskrypcji. Publikujemy raport w wybranym obszarze roboczym PowerBI i już po chwili mamy do dyspozycji następujące raporty:
{: style="text-align: justify;"}

- **At a glance** - pokazujący podstawowe dzienne statystyki użycia usługi w perspektywie ostatniego tygodnia. Na raporcie znajdziemy ilość wywołań, błędów i czasy odpowiedzi.
{: style="text-align: justify;"}
- **API Calls** - statystyki wywołań w rozbiciu na API i produkty. Mamy tu możliwość analizy czasu wykonania, regionu z którego wywoływane były żądania. Wszystko to z możliwością filtrowania po API, operacji, produkcie i subskrybentach.
{: style="text-align: justify;"}
- **Errors** - jak sama nazwa wskazuje, w tym raporcie mamy sporo informacji o zarejestrowanych błędach, w tym możliwość analizy ostatnich problemów z identycznym filtrowaniem jak powyżej.
{: style="text-align: justify;"}
- **Call Frequency** - zawiera informacje przetworzone przez Machine Learning.  Statystki obrazują wzorce użycia Twoich API, a na ich podstawie możesz zidentyfikować użytkowników (po IP lub subskrypcji), którzy odstają od normy. Może próbują się włamać, nadużywają usługę, a może mamy do czynienia z botem, który zbiera z naszych API wszelkie dane.
{: style="text-align: justify;"}
- **Relationships** - kolejny ciekawy raport, tym razem obrazujący relacje pomiędzy wywołani. Może któreś z Twoich API wywołuje grupę zależnych API, a te jeszcze coś innego. Tutaj masz okazję poszukać wzorców i popracować nad optymalizacją, a w zasadzie to odezwać się do twórców API, by zoptymalizowali schemat użycia.
{: style="text-align: justify;"}

Mała uwaga, jeśli Twoje APIM nie jest jakoś mocno używane, będziesz musiał(a) trochę poczekać na dane, które mają się pojawić w raportach. Pamiętaj również o ustawieniu odświeżania w źródle danych Twojego raportu PowerBI.{: style="text-align: justify;"}

### Jak działa wdrożone rozwiązanie?

Na poniższym, nieco uproszczonym diagramie, możemy zobaczyć jak statystyki API Management są zbierane i analizowane przez wdrożone rozwiązanie.
{: style="text-align: justify;"}

![Architektura rozwiązania](/assets/images/2017/11/api-management-1024x352.png)

W pierwszym kroku wszystkie zdarzenia wrzucane są do <a href="https://azure.microsoft.com/en-us/services/event-hubs/" target="_blank" rel="noopener">Event Huba</a>, ten z kolei wpięty jest jako wejście do <a href="https://azure.microsoft.com/en-us/services/stream-analytics/" target="_blank" rel="noopener">Stream Analytics</a>. Czemu nie bezpośrednio, otóż taka jest natura Stream Analytics. Potrafi konsumować tylko z IoT/Event Hub, lub z Blob Storage, ale to bardziej pod potrzeby wcześniej zgromadzonych danych. Usługa rozdziela spływające zdarzenia na poszczególne kategorie i umieszcza je w bazie <a href="https://azure.microsoft.com/en-us/services/sql-database/" target="_blank" rel="noopener">SQL Database</a>. Dla osób z nieco głębszym portfelem, lub przy naprawdę sporym ruchu, mamy możliwość zastąpienia bazy przez <a href="https://azure.microsoft.com/en-us/services/analysis-services/" target="_blank" rel="noopener">Azure Analysis Services.</a>
{: style="text-align: justify;"}

Dane wylądowały w bazie, ale dopiero tutaj zaczyna się prawdziwa zabawa. Są one uzupełniane (w oparciu o harmonogram) przez 4 <a href="https://azure.microsoft.com/en-us/services/logic-apps/" target="_blank" rel="noopener">Logic App</a>, 3 <a href="https://azure.microsoft.com/en-us/services/functions/" target="_blank" rel="noopener">Azure Functions</a> i wspomniany wcześniej <a href="https://azure.microsoft.com/en-us/overview/machine-learning/" target="_blank" rel="noopener">Azure Machine Learning</a>.  Przykładowo Logic App wykonują następujące operacje:
{: style="text-align: justify;"}

- Przeliczenie adresów IP z żądania na koordynaty geograficzne
- Analiza ostatnich wywołań w Machine Learning w oparciu o transfotmatę Furiera - ale mądrze zabrzmiało
- Wyznaczenie relacji pomiędzy wywołaniami
- Wgranie bazy danych adresów IP z www.maxmind.com

Ostatnia ważna sprawa to krótkie wyjaśnienie sposobu w jaki statystyki API Management są przekazywane do Event Hub. Wykorzystywana jest tu globalna polityka na bramie/proxy naszego APIM. Ma ona następującą postać (zamiast xxx powinna być nazwa Twojego Event Hub).
{: style="text-align: justify;"}

```xml
  <inbound>
    <log-to-eventhub partition-id="0" logger-id="xxx">
            @{
                
                var name = "";
                if(context.User != null)
                {
                    name = context.User.FirstName + " " + context.User.LastName;
                };
                
                var subId = "0";
                if(context.Subscription != null)
                {
                    subId = context.Subscription.Id;
                };
                
                var title = "CreatedDate,ServiceName,RequestId,IPAddress,Operation,OperationId,Api,ApiId,Product,ProductId,SubscriptionName,SubscriptionId,Length,Type";
                var values = string.Join(",", DateTime.UtcNow.ToString("o"), 
                                                context.Deployment.ServiceName, 
                                                context.RequestId, 
                                                context.Request.IpAddress, 
                                                context.Operation.Name,
                                                context.Operation.Id,
                                                context.Api.Name,
                                                context.Api.Id,
                                                context.Product.Name,
                                                context.Product.Id,                                                
                                                name, 
                                                subId,
                                                context.Request.Headers.GetValueOrDefault("Content-Length", "0"),
                                                "Request");
                return title + "\r\n"+ values;
            }
        </log-to-eventhub>
  </inbound>
  <backend>
    <forward-request follow-redirects="true" />
  </backend>
  <outbound>
    <log-to-eventhub partition-id="1" logger-id="xxx">
            @{
                var title = "CreatedDate,ServiceName,RequestId,StatusCode,StatusReason,Length,Type";
                var values = string.Join(",", DateTime.UtcNow.ToString("o"), 
                                                context.Deployment.ServiceName, 
                                                context.RequestId, 
                                                context.Response.StatusCode.ToString(),
                                                context.Response.StatusReason,
                                                context.Response.Headers.GetValueOrDefault("Content-Length", "0"),
                                                "Response");
                return title + "\r\n"+ values;
            }
        </log-to-eventhub>
  </outbound>
  <on-error>
    <log-to-eventhub partition-id="2" logger-id="xxx">
            @{
                var title = "CreatedDate,ServiceName,RequestId,Source,Reason,Message,Type";
                var values = string.Join(",", DateTime.UtcNow.ToString("o"), 
                                                context.Deployment.ServiceName, 
                                                context.RequestId, 
                                                context.LastError.Source,
                                                context.LastError.Reason,
                                                context.LastError.Message,
                                                "Error");
                return title + "\r\n"+ values;
            }
        </log-to-eventhub>
  </on-error>
</policies>
```

### Jakie są moje spostrzeżenia i co wymaga poprawy?

Pewnie Cię to odrobinę zdziwi, ale rozwiązanie nie spełniło wszystkich moich oczekiwań. Poniżej kilka spostrzeżeń:
{: style="text-align: justify;"}

- Największym minusem jest to, że wdraża się ono w jednym z regionów w USA, a ja mam swój APIM w Europie. Po analizie szablonu ARM postanowiłem zainstalować go w istniejącej grupie zasobów (West Europe), licząc, że zasoby odziedziczą z niej lokalizację. Wynik to połowa zasobów w Europie, a druga połowa w USA. Na więcej zabawy nie znalazłem czasu, ale zgłosiłem swoje żale do autorów 🙂
{: style="text-align: justify;"}
- Z racji ilości użytych komponentów koszt rozwiązania powinien zmieścić się w granicach 10$ za dzień, a to moim zdaniem całkiem sporo.
{: style="text-align: justify;"}
- Jestem pedantem jeśli chodzi o <a href="https://grabarz.pl//governance/standardy-nazewnicze-zasobow-azure/" target="_blank" rel="noopener">nazewnictwo zasobów</a>. Tutaj można mieć wrażenie, że każdy z elementów nazywała inna osoba.
{: style="text-align: justify;"}
- Raport PowerBI po kilku dniach przestał się wyświetlać, ponowna publikacja nie pomogła. Pewnie to wina PowerBI i mógłbym to zgłosić, ale tu moja przygoda z rozwiązaniem się skończyła (głównie z powodu trzech poprzednich faktów).
{: style="text-align: justify;"}

Podczas usuwania rozwiązania, trzeba **pamiętać o posprzątaniu globalnej polityki** w APIM. Ja nie posprzątałem i efekt był raczej opłakany, na szczęście nic na produkcji nie ucierpiało.  Domyślna polityka globalna ma następującą postać:
{: style="text-align: justify;"}

```xml
<policies>
  <inbound />
  <backend>
    <forward-request />
  </backend>
  <outbound />
</policies>
```

### Podsumowanie

Rozwiązanie jest w moim odczuciu bardzo fajne, ale jak widać nie da wszystkich. Moim zdaniem warto jest chociażby się mu przyjrzeć dla samej nauki architektury w Azure. Możesz też oczywiście je przerobić, lub stworzyć coś swojego i tańszego wykorzystując trik z polityką globalną.
{: style="text-align: justify;"}

Dla tych bardziej zapracowanych nie pozostaje nic innego, jak poczekać na integrację API Management z Application Insights. Krążą plotki, że może się ona już niedługo pojawić.
{: style="text-align: justify;"}

Photo via [VisualHunt](https://visualhunt.com/re/52c55e)