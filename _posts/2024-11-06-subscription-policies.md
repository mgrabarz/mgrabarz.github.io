---
title: Azure Subscription Policies - czyli jak (nie) ukraść subskrypcji Azure z firmy
date: 2024-11-06T00:00:01+01:00
header:
  teaser: /assets/images/2024/11/subscription-policies.png
categories:
  - Security
tags:
  - Security
  - Governance
  - Azure
  - Subscription
  - Policy
---

Z czym kojarzy Ci się Azure Subscription Policy? Zgaduję, że pierwsza myśl to klasyczne [Azure Policies](https://learn.microsoft.com/en-us/azure/governance/policy/overview), które w różnych trybach audytują, blokują czy też rekonfigurują niektóre zasoby w chmurze Microsoftu.

Azure Subscription Policies to coś zupełnie innego. Mało kto o nich słyszał, jeszcze mniej osób próbowało używać, a wdrożone widziałem tylko w jednej firmie - od tego czasu staram się to zmieniać. Poradniki na temat wdrażania Landing Zone o nich jedynie wspominają, a ryzyko jest spore. Tym bardziej, że jedno z typowych podejść z CAF (Cloud Adoption Framework) promujące demokratyzację chmury, czyli subskrypcja per rozwiązanie, mocno nas na ryzyko eksponuje.

### Azure Subscription Policies - co to takiego?

W odróżnieniu od zwykłych polityk, znajdziesz je w Portalu w zakładce [Subscriptions](https://portal.azure.com/#view/Microsoft_Azure_Billing/SubscriptionsBladeV2). Na górze w menu w "Advanced options" w "Manage Policies". Nie ma ich wiele, w zasadzie dwie:

- Pozwól Ownerom subskrypcji wynosić je do innych tenantów Entra ID (domyślnie zezwolone)
- Pozwól członkom Twojego tenanta Entra ID przynosić subskrypcje (domyślnie zezwolone).

Od razu odpowiem, że obie te operacje powinny być zabronione. Zmiany może wykonać jedynie Global Administrator tenanta jednocześnie podając krótką listę osób, dla których przenoszenie subskrypcji NIE będzie zablokowane.

### Jadę na ustawieniach domyślnych - co mi grozi?

Załóżmy czysto hipotetycznie, że LZ wdrażał w Twojej firmie jeden ze wspaniałych #GSI. Twoja firma zapłaciła miliony złotych monet za wdrożenie [gotowca z Github](https://github.com/Azure/Enterprise-Scale), macie Azure Policy, Management Grupy, Defendery, zbieracie audyt logi do Log Analytics, a wszystko leci do SOCa. Wypas, takich GSI to ja lubię... Właściciel rozwiązania dostał Ownera na subskrypcji, ale nic Wam nie grozi, przecież Azure Policy nie pozwalają wdrożyć niczego bez Private Endpointów, a wszystko jest w pełni monitorowane. Czy na pewno?

Wystarczy że wspomniany Owner stworzy sobie nowego, bezpłatnego tenanta Entra ID, na którym będzie Global Adminem (musi zrobić to kontem które jest Ownerem na sub - tego chyba nie muszę tłumaczyć). Wchodzi na subskrypcję, klika guzik Move, podaje TenantId i już, subskrypcja jest u niego. Co się wydarzyło?

- Wszystkie Azure Policy spływające dzielnie z Management Group przestały działać. Delikwent może wypuścić wszystkie bazy, data lejki, super wrażliwe dane na świat i mamy gotowy scenariusz eksfiltracji danych firmowych.
- Wszelki monitoring, audyt logi - nic nie działa, zostało w starym tenancie.
- Bez ograniczeń może postawić infrastrukturę za miliony, ile tylko quota i region pozwolą.
- Pewnie zastanawiasz się jaki zysk z powyższego, przecież teraz on/ona zapłaci... Nic bardziej mylnego - zmienił się tenant, ale BillingAccountID zostało stare. Twoja firma za to płaci, a ty "bawisz" się bez jej wiedzy. Może za miesiąc lub kwartał, jeśli mają niezły FinOps, coś zauważą.
- Możesz też namieszać w infrastrukturze i "oddać" subskrypcję z powrotem - patrz drugi Azure Subscription Policy. Czyli bez kontroli Azure Policy wdrożyć jakieś VMki z publicznymi adresami IP, zmienić routing omijając Azure Firewall, oddać subskrypcję do firmowego tenanta i wysłać zgłoszenie ServiceNow, że "się vNet peering rozpiął".

Jestem w stanie wymyślić jeszcze kilka innych paskudnych scenariuszy, jak wjeżdżanie do on-premises z takich zasobów, ale myślę, że powyższe przykłady już Cię przekonały. Po co straszyć!

### Jak wdrożyć Azure Subscription Policy z kodu?

Dokumentacja o tym milczy, przecież nikt tego nie ustawia... Jest na szczęście dokumentacja Azure Resource Management - uwaga, API jest już 3 lata w GA.

```bicep
resource subscriptionPolicy 'Microsoft.Subscription/policies@2021-10-01' = {
  name: 'default'
  scope: tenant()
  blockSubscriptionsIntoTenant: bool
  blockSubscriptionsLeavingTenant: bool
  exemptedPrincipals: [
    'string'
  ]
}
```

Scope tenant, uprawnienia Global Admina - chyba Wam zostawię jak to zautomatyzować.

Tymczasem dbajcie o bezpieczeństwo Azure!
