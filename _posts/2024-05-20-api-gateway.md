---
title: API Gateway
date: 2024-05-20T00:00:00+01:00
header:
  teaser: /assets/images/2024/05/ai-gateway.gif
categories:
  - API
tags:
  - API
  - APIM
  - OpenAI
  - Governance
  - AI
---

### Jak zintegrować Azure **#APIManagement** z Azure **#OpenAI**?

W dzisiejszym świecie AI, bezpieczeństwo i zarządzanie stają się kluczowe.

Chcę Ci pokazać, jak Azure API Management (APIM) może pomóc Ci w wykorzystaniu usług Azure OpenAI w sposób bezpieczny i efektywny.

🔶 Pierwszym krokiem jest zrozumienie, że APIM to nie tylko brama do Twoich API, ale także potężne narzędzie do monitorowania, ograniczania ruchu i zabezpieczania Twoich punktów końcowych. Dzięki APIM, możesz kontrolować dostęp do Azure OpenAI, zarządzać nim i skalować według potrzeb.

🔶 Drugi element, to gotowość Twojej organizacji do zarządzania API. Jak podchodzisz do zagadnienia, czy wdrożyliście APIMa oraz zasady API Governance, o których pisałem w poprzednim [poście](https://lnkd.in/dNfGAQBn)?

🔶 Trzeci to import OpenAI ze swagger do Twojego APIM. Linki do poszczególnych wersji API i swaggera w komentarzu.

### Co dalej?

W kolejnych wpisach szczegółowo omówię kilka kluczowych scenariuszy wykorzystania APIM dla OpenAI. Są to między innymi:

- **Scenariusze autoryzacji i delegowania uprawnień do Azure OpenAI.**

- **Ograniczanie liczby zapytań (rate-limiting)** - elastyczna kontrola ruchu, pozwalająca na dostosowanie limitów w zależności od tożsamości klienta, poziomu subskrypcji czy produktu API.

- **Balansowanie ruchu** - rozdzielenie ruchu między instancjami Azure OpenAI, w zależności od ich regionu, dostępności lub obciążenia.

- **Circuit Breaking** - jak reagować nie niedostępność usługi lub zamokować jej funkcjonalność.

- **Zarządzanie kosztami i monitorowanie użycia** - jak efektywnie kontrolować wysyłane zapytania, logować je i zapewnić odpowiedzialność za koszty.

Zapraszam Cię do [śledzenia](https://lnkd.in/dDgccHWR) mojergo LinkedIn i odkrywania, jak APIM może stać się Twoim sprzymierzeńcem w świecie AI. Do zobaczenia w następnych postach!
