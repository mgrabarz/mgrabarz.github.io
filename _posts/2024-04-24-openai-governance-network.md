---
title: OpenAI - Konfiguracja bezpieczeństwa - Sieć
date: 2024-04-24T23:00:00+01:00
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

Jak poprawnie skonfigurować i zabezpieczyć usługę Azure OpenAI?

Dobrym punktem wyjścia są wytyczne i kontrolki zdefiniowane w ramach Microsoft Cloud Security Benchmark v1. MCSB to zestaw najlepszych praktyk i zaleceń opracowany przez Microsoft, który pomaga w poprawie bezpieczeństwa wdrażanych aplikacji, danych i usług w środowisku chmurowym. Co istotne, kontrolki mogą być łatwo zmapowane na rynkowe standardy, takie jak CIS v8, NIST SP 800-53 r4 lub PCI-DSS v3.2.1

W dzisiejszym wpisie na warsztat biorę pierwszy, niezwykle istotny obszar - NS (Network security).

Co można i należy skonfigurować w obszarze konfiguracji sieciowej, które z kontrolek mają kluczowe znaczenie w przypadku Azure OpenAI?

🔶 NS-1: Ustalenie granic segmentacji sieci

Usługa Azure OpenAI to w pewnym sensie SaaS (Software as a Service) i nie ma możliwości wdrożenia jej do delegowanej podsieci w ramach sieci wirtualnej w Azure.

🔶 NS-2: Kontrola ruchu przychodzącego

W tym obszarze mamy do zrobienia dwie rzeczy:
Wdrażamy Private Endpoint wskazujący na instancję usługi, dzięki czemu umożliwiamy komunikację z poziomu sieci prywatnej Azure.
Wyłączamy publiczny dostęp sieciowy do usługi albo poprzez przełącznik “Disable Public Network Access”, albo poprzez skonfigurowanie reguł firewall wewnątrz usługi.

W niektórych organizacjach, w których poszczególni użytkownicy mają szerokie prawa administracyjne do zasobów Azure należy rozważyć wdrożenie polityk bezpieczeństwa:

- 𝗔𝘇𝘂𝗿𝗲 𝗔𝗜 𝗦𝗲𝗿𝘃𝗶𝗰𝗲𝘀 𝗿𝗲𝘀𝗼𝘂𝗿𝗰𝗲𝘀 𝘀𝗵𝗼𝘂𝗹𝗱 𝗿𝗲𝘀𝘁𝗿𝗶𝗰𝘁 𝗻𝗲𝘁𝘄𝗼𝗿𝗸 𝗮𝗰𝗰𝗲𝘀𝘀 w trybie Deny - polityka blokuje wszystkie wdrożenia (i zmiany) Azure OpenAI, w których dostęp publiczny jest włączony i reguły firewall nie mają ustawionej domyślnej reguły Deny.
- 𝗖𝗼𝗴𝗻𝗶𝘁𝗶𝘃𝗲 𝗦𝗲𝗿𝘃𝗶𝗰𝗲𝘀 𝘀𝗵𝗼𝘂𝗹𝗱 𝘂𝘀𝗲 𝗽𝗿𝗶𝘃𝗮𝘁𝗲 𝗹𝗶𝗻𝗸 - sprawdza czy instancja usługi używa Private Endpointa.

Niestety z racji problemów z np Terraform nie należy wdrażać powyższej polityki w trybie Deny (zresztą wymagałoby to jej przepisania).

Aby dowiedzieć się więcej o funkcjach, które można włączyć lub wyłączyć, zachęcam do zapoznania się z kolejnym postem i obserwowania mojego [profilu](https://lnkd.in/gHjt9eSb)
