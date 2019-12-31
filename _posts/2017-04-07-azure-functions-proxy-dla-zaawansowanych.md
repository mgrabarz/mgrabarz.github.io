---
title: Azure Functions Proxy dla zaawansowanych
date: 2017-04-07T05:55:14+02:00
image: /assets/images/2017/04/07-04-2017-functions1.png
categories:
  - Functions
tags:
  - Azure
  - Functions
  - Proxy
---
Od pewnego czasu w Azure Functions, mojej ulubionej usłudze w chmurze, dostępna jest możliwość tworzenia Proxies, w dniu pisania tego posta funkcjonalność ta pozostaje w Public Preview. Kilka słów na temat tworzenia proxy można przeczytać w dokumentacji <a href="https://docs.microsoft.com/en-us/azure/azure-functions/functions-proxies" target="_blank" rel="noopener noreferrer">tutaj</a>. Po przeczytaniu dokumentacji można odnieść wrażenie, że cała funkcjonalność ogranicza się do bardzo podstawowego rewrite i polega na podaniu matching condition (metoda HTTP i route template) oraz backend uri, do którego zostanie przepisany request.

### Kilka sztuczek w proxies.json

Okazuje się, że plik proxies.json powstający podczas konfiguracji funkcji przez portal ma nieco szerszą składnię, oto kilka przydatnych dodatków:

  * W związku z tym, że Functions mają wiele wspólnego z klasycznym AppService, możemy korzystać z typowych dla AppService <a href="https://github.com/projectkudu/kudu/wiki/Azure-runtime-environment" target="_blank" rel="noopener noreferrer">zmiennych środowiskowych</a>, w związku z tym użycie %WEBSITE_HOSTNAME% w backend uri jest w zupełności poprawne.
  * Cytowana wcześniej dokumentacja wspomina również o użyciu ApplicationSettings w podobny sposób, bardzo polecam!
  * Zamiast wymieniać wszystkie wspierane HTTP verbs (GET, POST, HEAD, PUT, DELETE, OPTIONS, PATCH, CONNECT, TRACE), możemy zwyczajnie wpisać ANY.
  * Możemy dodać opisy proxy, używając &#8220;desc&#8221; <pre class="EnlighterJSRAW" data-enlighter-language="json">"My Proxy":{
 "desc":[
 "Simple proxy description :)"
 ],
 "matchCondition":{
 "methods":[
 "GET"
 ],
 "route":"/{id}"
 },
 "backendUri":"https://%WEBSITE_HOSTNAME%/api/Persons/{id}"
}</pre>

  * Mamy również opcję dokonywania zmian w Request i w Response, ale o tym poniżej.

### Modyfikacja wartości Request i Response

Zmiany w strukturze wiadomości przychodzącej maja na celu przystosowanie jej przed przepisaniem do docelowego adresu Url. Modyfikacja wiadomości wychodzącej może z kolei służyć ukryciu wewnętrznej implementacji naszych usług.

Przepisywanie dokonywane jest poprzez dodanie &#8220;requestOverrides&#8221; lub &#8220;responseOverrides&#8221; w definicji naszej funkcji (w json). Możemy również czytać i przekazywać takie elementy jak nagłówki wiadomości, wartości query string, status odpowiedzi a nawet jej body. Poniższy fragment kodu obrazuje kilka z możliwych operacji:

<pre class="EnlighterJSRAW" data-enlighter-language="json">"Override proxy":{
   "matchCondition":{
      "methods":[
         "GET"
      ],
      "route":"/clients/{uid}"
   },
   "backendUri":"https://otherfunctionapp/api/clients/{uid}/{request.querystring.operation}",
   "requestOverrides":{
      "backend.request.querystring.newkey":"abc",
      "backend.request.querystring.userId":"{uid}",
      "backend.request.querystring.token":"%FUNCTION_TOKEN%",
      "backend.request.header.sourceapp":"%MY_APP_NAME%",
      "backend.request.header.apiversion":"request.querystring.version"
   },
   "responseOverrides":{
      "response.header.apiversion":"backend.response.header.version",
      "response.header.anotherHeader":"Some other value is %CUSTOM_SETTING%",
      "response.status":"200 OK"
   }
}</pre>

Możemy również definiować wyrażenia zwracające różne wartości w zależności od parametrów:

<pre class="EnlighterJSRAW" data-enlighter-language="json">"responseOverrides":{
   "backend.response.statusCode":{
      "302":{
         "response.header.Location":"http://myNewlocation"
      },
      "501":{
         "response.header.appversion":"backend.response.header.version",
         "response.status":"404"
      },
      "2XX":{
         "response.header.mystatus":"Done!"
      },
      "4xx":{
         "response.status":"{backend.response.statusCode} Not my fault",
         "response.body":"Valuefrom backend = {backend.response.statusCode}."
      }
   }
}</pre>

### Mokowanie API przy uzyciu Proxies

Chwilę wcześniej wspomniałem, że możemy również modyfikować body w wiadomościach, to otwiera kilka ciekawych możliwości. Jedną z nich jest proste mokowanie zwracanych wiadomości w naszym API:

<pre class="EnlighterJSRAW" data-enlighter-language="json">"Mock API Status":{
   "matchCondition":{
      "methods":[
         "GET"
      ],
      "route":"status/{code}/{desc}"
   },
   "responseOverrides":{
      "response.status":"{code} {desc}"
   }
}</pre>

To z kolei, wzbogacone o zmiany w body, umożliwia tworzenie prostych mock&#8217;ów symulujący zwracane wartości w XMl lub w json.

<pre class="EnlighterJSRAW" data-enlighter-language="json">"Json response":{
   "matchCondition":{
      "methods":[
         "GET"
      ],
      "route":"json/{id}"
   },
   "responseOverrides":{
      "response.status":"200 OK",
      "response.header.Content-Type":"application/json",
      "response.body":{
         "User":{
            "Id":"{id}",
            "Name":"User Name {id}"
         }
      }
   }
},
"XML response":{
   "matchCondition":{
      "methods":[
         "GET"
      ],
      "route":"xml/{id}"
   },
   "responseOverrides":{
      "response.status":"200 OK",
      "response.header.Content-Type":"application/xml",
      "response.body":"&lt;xml&gt;&lt;User&gt;&lt;Id&gt;{id}&lt;/Id&gt;&lt;Name&gt;User Name {id}&lt;/Name&gt;&lt;/User&gt;&lt;/xml&gt;"
   }
}</pre>

### Podsumowanie

Azure Functions Proxy dzięki opisanym powyżej mechanizmom staje się bardzo elastycznym rozszerzeniem. Całkiem realne wydaje się stworzenie np. prostego serwera http, który przepisuje url żądania na wewnętrzny parametr funkcji, ta z kolei w oparciu o parametr czyta i zwraca plik ze Azure Storage Blob, a to wszystko prawie za darmo.

Przytoczone przeze mnie przykłady użycia Proxies częściowo pochodzą z wewnętrznego forum Azure Advisors. Jeżeli macie możliwość się tam dostać, z pewnością będziecie o krok bliżej do zaawansowanych przykładów, zdecydowanie warto.