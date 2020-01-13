---
title: Wyłączanie walidacji maila w Azure AD B2C
date: 2017-03-28T06:19:51+02:00
header:
  teaser: /assets/images/2017/03/b2c.png
categories:
  - B2C
tags:
  - Azure
  - B2C
  - Email
---
**Azure Active Directory B2C** to globalna, wysokodostępna usługa chmurowa służąca do zarządzania tożsamością klientów. B2C jest bardzo podobne do szeroko używanego Azure Active Directory (AAD) i na nim się opiera. Przy pomocy B2C możemy uwierzytelniać naszych klientów poprzez klasyczny login/email, Microsoft LiveID, ale również poprzez media społecznościowe takie jak Facebook czy Google+. B2C ma również swoją wersję **Premium**, dostępną aktualnie w private preview, jeżeli Cię ten temat interesuje, daj mi proszę znać w komentarzach.
{: style="text-align: justify;"}

## Proces zakładania konta i walidacja emaila

Przy standardowo skonfigurowanym procesie rejestracji użytkowników, kiedy decydują się oni użyć modelu login/hasło (przy zewnętrznych dostawcach jak Facebook jest inaczej), wymagana jest weryfikacja poprawności adresu email. Jest to szczególnie ważne ponieważ właśnie adres email jest identyfikatorem użytkownika i nie ma możliwości jego zmiany w późniejszym czasie.
{: style="text-align: justify;"}

Proces przebiega w dwóch krokach:

<img class="alignnone size-full wp-image-217" src="http://marek.grabarze.com/wp-content/uploads/2017/03/2017-03-28-06_44_05-b2c-sendcode.png" alt="2017-03-28 06_44_05-B2C - SendCode" width="679" height="450" srcset="https://marekgrabarz.pl/wp-content/uploads/2017/03/2017-03-28-06_44_05-b2c-sendcode.png 679w, https://marekgrabarz.pl/wp-content/uploads/2017/03/2017-03-28-06_44_05-b2c-sendcode-300x199.png 300w" sizes="(max-width: 679px) 100vw, 679px" /> 

Na wskazany email dostajemy kod weryfikacyjny, który musimy wprowadzić do formularza.

<img class="alignnone size-full wp-image-216" src="http://marek.grabarze.com/wp-content/uploads/2017/03/2017-03-28-06_44_05-b2c-entercode.png" alt="2017-03-28 06_44_05-B2C - EnterCode" width="520" height="272" srcset="https://marekgrabarz.pl/wp-content/uploads/2017/03/2017-03-28-06_44_05-b2c-entercode.png 520w, https://marekgrabarz.pl/wp-content/uploads/2017/03/2017-03-28-06_44_05-b2c-entercode-300x157.png 300w" sizes="(max-width: 520px) 100vw, 520px" /> 

Proces wydaje się być bardzo prosty, szczególnie dla informatyków i specjalistów od chmury, ale niekoniecznie dla użytkowników końcowych. Szczególnie trudno przebiega on w przypadku osób starszych, które nie mają styczności z tego typu serwisami na co dzień. Trudności są następujące:
{: style="text-align: justify;"}

- Formularz dostępny tylko w języku angielskim
- Email potwierdzający podpisany jako Microsoft i często wpadający do spamu 😉
- Treść emaila po angielsku
- Potrzeba przepisania kodu, zamiast kliknięcia w link

## Wyłączenie walidacji emaila

W zeszłym miesiącu na blogu nowości Azure pojawił się następujący wpis: <a href="https://azure.microsoft.com/en-us/blog/new-in-azure-ad-b2c/" target="_blank" rel="noopener noreferrer">What's new in Azure Active Directory B2C</a>, a w nim sekcja "Friction-free consumer sign-up". Okazuje się, że możliwość wyłączenia walidacji istniała w B2C od dłuższego czasu, ale dopiero w lutym zostało to opisane i ogłoszone na blogu.
{: style="text-align: justify;"}

Ustawienie to jest dość mocno ukryte, ale z pomocą powyższego wpisu łatwe do odnalezienia. Nawigujemy do ustawień naszej polisy, dostosowywania UI, dalej ustawienia lokalnej strony zakładania konta, klikamy w atrybut Email Address i jesteśmy na miejscu, uff!

<img class="alignnone size-full wp-image-218" src="http://marek.grabarze.com/wp-content/uploads/2017/03/2017-03-28-06_47_20-b2c-settings.png" alt="2017-03-28 06_47_20-B2C- Settings" width="1199" height="398" srcset="https://marekgrabarz.pl/wp-content/uploads/2017/03/2017-03-28-06_47_20-b2c-settings.png 1199w, https://marekgrabarz.pl/wp-content/uploads/2017/03/2017-03-28-06_47_20-b2c-settings-300x100.png 300w, https://marekgrabarz.pl/wp-content/uploads/2017/03/2017-03-28-06_47_20-b2c-settings-768x255.png 768w, https://marekgrabarz.pl/wp-content/uploads/2017/03/2017-03-28-06_47_20-b2c-settings-1024x340.png 1024w" sizes="(max-width: 1199px) 100vw, 1199px" /> 

Przełączamy "Require verification" na "no" i wyłączamy sprawdzanie emaila, pytanie tylko czy jest to najlepsze rozwiązanie? Uwaga, nie zapomnij zapisać zmian na "Edit policy".

## Konsekwencje wyłączonej walidacji

Przy wyłączonej walidacji narażamy się na sytuację, że konto założy u nas praktycznie każdy, bez żadnej kontroli (konto z nieistniejącym emailem, podszywanie się pod email innych osób). Dodatkowo w procesie rejestracji nasi przyszli klienci mogą zrobić literówkę w swoim adresie mailowym i nigdy się nie dowiedzą co poszło nie tak. Jest to szczególnie niebezpieczne podczas rejestracji przez urządzenia mobilne, pole email nie ma tu odpowiednich atrybutów, przez co przywołuje klasyczną klawiaturę (bez małpki i z włączonym słownikiem).
{: style="text-align: justify;"}

**Bezwzględnie musimy dokonać walidacji emaila**, ale pozostaje to teraz na naszych barkach. Tutaj mamy pełną swobodę, email w odpowiedniej wersji językowej, link, dodatkowy formularz - pewnie z miejscem na kartę kredytową 🙂
{: style="text-align: justify;"}

Możemy również skorzystać z GraphAPI i przeprowadzić użytkownika przez proces rejestracji według naszego uznania, z pominięciem GUI B2C, ale to już zupełnie inny temat na wpis.
{: style="text-align: justify;"}