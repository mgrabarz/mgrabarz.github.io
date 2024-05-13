---
title: OpenAI - Contnet Filters
date: 2024-04-17T23:00:00+01:00
header:
  teaser: /assets/images/2024/04/openai-data.jpeg
categories:
  - API
tags:
  - OpenAI
  - AI
  - Security
  - DataSecurity
  - Governance
---

Masz wątpliwości, czy wdrożyć hashtag#Azure hashtag#OpenAI z obawy przed dostępem Microsoftu do Twoich danych?

W hashtag#Protopia często pracujemy dla takich branż jak bankowa czy ubezpieczeniowa. Częścią naszej pracy jest przełamywanie obaw dotyczących tego zagadnienia.

Niewiele osób wie, że odpowiednie wdrożenie OpenAI może zapewnić, że dane nigdy nie będą w żaden sposób używane przez Microsoft. Można się przed tym skutecznie i w pełni zabezpieczyć. Według dokumentacji, sprawa wygląda dość jasno, jednak 𝗻𝗶𝗲𝘀𝘁𝗲𝘁𝘆 musimy zadbać o kilka dodatkowych szczegółów.

Jak to zrobić?
Po pierwsze należy włączyć Customer Lockbox, by wsparcie techniczne nie miało dostępu do usługi. Następnie, korzystając z OpenAI, musimy mieć świadomość o istnieniu dwóch funkcji: content filters i abuse monitoring.

- Content filters – wychwytuje zapytania niezgodne z prawem, przekleństwa, itp. Funkcja ta powinna być włączona.
- Abuse monitoring – sprawia, że jeżeli coś zostanie zatrzymane przez content filters, Microsoft poddaje to analizie, czyli między innymi human review, i niekiedy przekazuje do właściwych organów. Funkcja ta powinna być wyłączona.

Dlaczego?
To, że nasze dane wrażliwe, na przykład dane bankowe, ubezpieczeniowe czy dane osobowe, przekazywane w prompcie, są weryfikowane przez osoby czy firmy trzecie, oczywiście jest nie do zaakceptowania przez firmy z sektora regulowanego.

Dobra wiadomość jest taka, że abuse monitoring MOŻESZ wyłączyć. W interfejsie graficznym może nie być takiej opcji, ale można zgłosić tę kwestię do Microsoftu, wypełniając specjalny formularz (jeśli chcesz, bym Ci go przesłał, napisz w komentarzu).

I tutaj często pojawia się pytanie: ale jeśli wyłączę, to nie będę miał content filtering?

I właśnie nie, content filtering nadal będzie działał. To znaczy, zgodność z wymaganiami polskiego prawa w branży regulowanej możesz osiągnąć poprzez wyłączenie samej funkcji abuse monitoring, pozostawiając content filtering włączony.

Jakie są Twoje główne obawy związane z wdrażaniem AI w Twojej branży?

hashtag#Protopia jako Microsoft Advanced Specialisation Partner, pracujący dla największych firm z branży bankowej i ubezpieczeniowej, pomaga klientom zagwarantować bezpieczne wdrożenie usług AI.

Jeśli chcesz wiedzieć, jakie inne funkcje możesz włączyć lub wyłączyć, zapraszam do obserwowania mojego profilu: https://lnkd.in/gHjt9eSb
