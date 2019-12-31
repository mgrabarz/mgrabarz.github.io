---
title: Błąd wyświetlania metryk w Azure Storage
date: 2017-12-23T22:32:51+01:00
header:
  image: /assets/images/2017/12/11682625885_4e0bdb28d8_c.jpg
categories: [Storage]
tags: [ApplicationInsights, Azure, Error, Metrics, Storage]
---

Ostatnio na blogu niewiele się dzieje. Na szczęście wreszcie pokończyłem wszystkie dodatkowe sprawy, które na siebie wziąłem. Mam nadzieję, że w najbliższym czasie wreszcie bloga rozruszam.
{: style="text-align: justify;"}

Na rozgrzewkę postanowiłem podzielić się pewnym problemem i jego rozwiązaniem dotyczącym Azure Storage Account (ale nie tylko). Mianowicie chodzi o błąd wyświetlania metryk na Storage Account o następującej treści &#8220;**A problem occurred loading metrics. Please try again later.**&#8221;
{: style="text-align: justify;"}

<img class="alignnone wp-image-572 size-full" src="https://marekgrabarz.pl/wp-content/uploads/2017/12/2017-12-23-21_52_44.png" alt="A problem occurred loading metrics. Please try again later." width="758" height="512" srcset="https://marekgrabarz.pl/wp-content/uploads/2017/12/2017-12-23-21_52_44.png 758w, https://marekgrabarz.pl/wp-content/uploads/2017/12/2017-12-23-21_52_44-300x203.png 300w" sizes="(max-width: 758px) 100vw, 758px" /> 

### Rozwiązanie problemu

Warto zauważyć, że nie jest to błąd, który pojawił się raz. Po przejrzeniu wszystkich subskrypcji Azure (około 50) z którymi pracuję okazało się, że sporo Azure Storage Account ma taki właśnie problem.
{: style="text-align: justify;"}

Początkowo stwierdziłem, że błąd wynika z niepoprawnej konfiguracji diagnostyki na Storage Account. Próbowałem między innymi włączać ją i wyłączać, czy też próbować różnych kombinacji metryk i logowania. Niestety moje próby i poszukiwanie rozwiązania w sieci nie przyniosło żadnych efektów.
{: style="text-align: justify;"}

Postanowiłem w tym miejscu skorzystać z niezawodnego F12 (Debugger) w przeglądarce by sprawdzić jak wygląda komunikacja Portalu Azure z chmurą i na tej podstawie poszukać rozwiązania. Moim oczom ukazały się następujące błędy:
{: style="text-align: justify;"}

```
Error 400: https://insights1.exp.azure.com/insights/api/Insights/GetMetricHistoryCollection HTTPS POST 

{"message":"There was an error processing your request. Please try again in a few moments.","httpStatusCode":"BadRequest","xMsServerRequestId":null,"stackTrace":null}
```
```
Error 409: https://insights1.exp.azure.com/insights/api/Insights/ListMetricDefinitions HTTPS POST

{"message":"Please register the subscription 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' with Microsoft.Insights.","httpStatusCode":"Conflict","xMsServerRequestId":null,"stackTrace":null}
```

Z drugiego błędu wynika już bezpośrednio rozwiązanie problemu. Musimy zarejestrować Microsoft.Insights resource provider w naszej subskrypcji, szczegółową instrukcję znajdziesz <a href="https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-supported-services" target="_blank" rel="noopener">tutaj</a>.
{: style="text-align: justify;"}

### Podsumowanie

Moim zdaniem nieco dziwne jest, że ten provider nie jest rejestrowany podczas tworzenia nowego Azure Storage Account. Mamy też wyjaśnienie dlaczego błąd może pojawiać się stosunkowo często &#8211; wystarczy że nigdy nie używaliśmy Application Insights w ramach naszej subskrypcji.
{: style="text-align: justify;"}

Zabawne jest też to, że na podobny problem natrafiłem podczas włączania diagnostyki w API Management. Nie działała, do czasu aż ręcznie zarejestrowałem Microsoft.Insights resource provider.
{: style="text-align: justify;"}

Dzięki za lekturę. Życzę Wam wszystkim Wesołych Świąt i szczęścia w nowym roku!
{: style="text-align: justify;"}

Photo by [wwarby](https://visualhunt.com/author/531e57) on [Visualhunt](https://visualhunt.com/re/70457a) /  [CC BY](http://creativecommons.org/licenses/by/2.0/)
