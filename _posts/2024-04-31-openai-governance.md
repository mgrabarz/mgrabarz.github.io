---
title: OpenAI - Konfiguracja bezpieczeństwa #2
date: 2024-04-31T23:00:00+01:00
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

Jeśli myślisz, że poprawnie skonfigurowałeś Azure OpenAI w obszarze ochrony przetwarzania danych, to możesz się mylić, dopóki nie wdrożysz poniższych ustawień.

W poprzednim swoim wpisie mówiłem o konfigurację bezpieczeństwa OpenAI w kontekście sieci.

Dziś omówię konfigurację Azure Open AI w obszarze ochrony przetwarzania danych, dlatego, że wiele firm nie wie, jak zrobić to poprawnie.

Co jest niezwykle istotne:

🔶 DP-1: Klasyfikacja danych
Sama usługa nie przechowuje danych (poza szczególnymi przypadkami), a jedynie je przetwarza. Nie dysponuje zatem wbudowanymi funkcjami do klasyfikacji. Wystarczy zatem klasyfikować dane źródłowe przy pomocy na przykład Azure Purview.

🔶 DP-2: Kontrola ruchu wychodzącego
Zdecydowanie nie chcemy, by nasze dane zostały wykradzione lub przekazane do zewnętrznych usług. Dlatego podczas wdrażania usługi włącz kontrolę ruchu wychodzącego (restrictOutboundNetworkAccess=true), a następnie stwórz listę adresów (nazwy lub IP), z którymi może ona się komunikować (allowedFqdnList). Ustawienia te nie są dostępne z poziomu interfejsu użytkownika.

🔶 DP-3: Szyfrowanie komunikacji
Cała komunikacja z OpenAI powinna „lecieć po HTTPS”. Na szczęście szyfrowanie jest zawsze włączone i to w wersji TLS 1.2+.

🔶 DP-4: Szyfrowanie danych w spoczynku
Podobnie jak w punkcie powyżej, funkcja jest domyślnie włączona, a wszystkie dane są szyfrowane za pomocą AES 256. Nie musisz zatem nic konfigurować (chyba że wymagają tego regulacje branżowe).

🔶 DP-5: Szyfrowanie własnymi kluczami (CMK)
No właśnie! Co z regulacjami, takimi jak na przykład regulacje KNF? Niestety, tutaj musimy się odrobinę postarać i dostarczyć własny klucz szyfrujący (czyli Customer Managed Key). Jak to zrobić? Wymagana jest usługa KeyVault, którą trzeba odpowiednio skonfigurować. Sam klucz podlega automatycznej rotacji, zgodnie z wymogami Twojej firmy.

PS. Jeśli chcesz wiedzieć, jakie inne funkcje możesz włączyć lub wyłączyć, zapraszam do obserwowania 👨‍💻 https://lnkd.in/dDgccHWR i zapoznania się z kolejnym postem.

Wiele ciekawych informacji znajdziesz też na profilach moich kolegów z zespołu PROTOPIA.

👨‍💻Profil Szymona: https://lnkd.in/dtB_mtXT

👨‍💻Profil Łukasza: https://lnkd.in/djKGAV9f

[Protopia](https://protopia.tech), jako Microsoft Advanced Specialization Partner, wspierający największe firmy z sektora bankowego i ubezpieczeniowego, pomaga klientom zapewnić bezpieczne wdrożenie usług AI.

Oryginalnie opublikowane na LinkedIn: https://www.linkedin.com/posts/grabarz_azure-openai-protopia-activity-7190994157744648193-TwYh
