---
title: OpenAI - Konfiguracja bezpieczeństwa #3
date: 2024-05-06T00:00:00+01:00
header:
  teaser: /assets/images/2024/04/openai.webp
categories:
  - API
tags:
  - OpenAI
  - AI
  - Security
  - DataSecurity
  - Governance
---

🔶 Jak poprawnie skonfigurować Azure OpenAI podczas wdrożenia?

W poprzednich dwóch wpisach omówiłem [ustawienia sieciowe](https://lnkd.in/dRi8NCTh) oraz [ochronę danych](https://lnkd.in/dafZS6_w).

Pozostało nam kilka istotnych elementów z pozostałych kategorii, o których należy pamiętać:

🔶 IM-1: Użycie EntraID na potrzeby uwierzytelniania

Domyślnie zalecam Ci używanie EntraID na potrzeby uwierzytelniania użytkowników i aplikacji. Uwierzytelnianie za pomocą kluczy powinno być domyślnie wyłączone i blokowane przez politykę.

Może się jednak zdarzyć, że biblioteki lub rozwiązania integrujące się z OpenAI nie potrafią posłużyć się logowaniem z EntraID. Wykonujemy wtedy analizę ryzyka i docelowo włączamy klucze jako odstępstwo od polityki.

🔶 IM-3: Użycie tożsamości zarządzanej (MSI) dla OpenAI

W tym obszarze sprawa jest prosta. Używamy MSI (Managed Identity) gdzie i kiedy się da, a jego włączenie powinno być wymuszane polityką.

🔶 PA-7: Zapewnienie dostępu administracyjnego do usługi

Dostęp konfigurujemy przy pomocy RBAC (role-based access control). Zwróć szczególną uwagę na te dwie wbudowane role:
Cognitive Services OpenAI User – odczyt i użycie modeli dostarczanych w ramach usługi, w tym dostęp do kluczy i adresów usługi.
Cognitive Services OpenAI Contributor – modyfikacja modeli, tworzenie własnych modeli i dostarczanie źródeł danych do trenowania modeli.

🔶 PA-8: Dostęp dla działów wsparcia Microsoft

Jeśli wymagają tego względy regulacyjne, włącz Customer Lockbox.

🔶 LT-3/4: Zbieranie logów

Skonfiguruj zbieranie logów poprzez Diagnostic Settings usługi i wysyłaj je do odpowiednich instancji Log Analytics. Warto pomyśleć o równoległym wysyłaniu logów do LA zespołu utrzymaniowego i LA działu bezpieczeństwa.
Jakie logi wysyłać – zarówno logi audytowe, jak też logi request response.
Niemal zapomniałem, do tego też przyda się polityka, by nic nie uciekło, kiedy zaczniemy te logi przeszukiwać.

PS. Zapraszam do obserwowania mojego profilu [👨‍💻](https://lnkd.in/dDgccHWR). Już niedługo opublikuję serię wpisów na temat zarządzania API oraz jak do tego podejść w kontekście usług AI z Microsoftu.

[Protopia](https://protopia.tech), jako Microsoft Advanced Specialization Partner, wspierający największe firmy z sektora bankowego i ubezpieczeniowego, pomaga klientom zapewnić bezpieczne wdrożenie usług AI.

Oryginalnie opublikowane na [LinkedIn](https://www.linkedin.com/posts/grabarz_azure-activity-7194250467713966080-oJpZ)
