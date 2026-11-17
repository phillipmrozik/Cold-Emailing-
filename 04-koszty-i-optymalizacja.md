# 04. Koszty i optymalizacja

Na początku system działał dobrze, ale był drogi: ok. 0,30 USD za jeden gotowy mail i ponad 30 tysięcy wykonań n8n miesięcznie, czyli dużo ponad limit planu. Poniżej opisuję, co zmieniłem.

## OpenAI: z ok. 0,30 USD do ok. 0,04 USD za mail

| Zmiana | Efekt |
|---|---|
| tryb `service_tier: flex` dla wszystkich zapytań, z automatycznym powtórzeniem w zwykłym trybie | ok. połowa ceny za te same tokeny |
| wyszukiwanie w internecie tylko wtedy, gdy maila nie ma na stronie | większość firm w ogóle go nie potrzebuje |
| wyszukiwanie z `search_context_size: low` i maks. 3 zapytaniami | dużo mniej tokenów z wyników |
| analiza i research na mniejszym modelu (`gpt-5.4-mini`), pisanie i kontrola na większym | jakość tam, gdzie ją widać |
| niski poziom rozumowania (`effort: low`) w kontrolerze | kontrola nie potrzebuje długiego myślenia |
| najpierw firmy ze stroną www | mniej researchu kończy się odrzuceniem |

Najważniejsze w tym wszystkim było to, że jakość maili się nie zmieniła, bo pisanie i kontrola zostały na mocniejszym modelu.

## n8n: z ok. 38 000 do ok. 4 000 wykonań miesięcznie

W n8n Cloud płaci się za wykonania (jedno uruchomienie workflowu = jedno wykonanie), więc liczyło się, jak często coś startuje.

| Zmiana | Efekt |
|---|---|
| wysyłka co 30 minut, do 4 maili w jednym przebiegu (pętla z przerwą 1–3 min) zamiast osobnego uruchomienia na każdy mail | kilka razy mniej wykonań |
| research tylko poza godzinami wysyłki: co 10 minut po południu, co godzinę w nocy i w weekendy | maile i tak są przygotowane z zapasem |
| skrzynka co 15 minut w dzień, co godzinę w nocy | w nocy prawie nikt nie odpisuje |
| weryfikacja adresów raz rano i co godzinę w czasie wysyłki | wystarczy przed każdą porcją |
| zapas gotowych maili na 2 dni (`bufor_dni`) | silnik nie pracuje na pusto, gdy kolejka jest pełna |

## Inne zabezpieczenia kosztów

- research jednej firmy ma limit prób, więc „trudna” strona nie zjada budżetu w kółko,
- raport dnia pokazuje stan kredytów Firecrawla, a przy niskim stanie przychodzi ostrzeżenie,
- alert o skończonych środkach w OpenAI przychodzi od razu, bez czekania na kolejne błędy.
