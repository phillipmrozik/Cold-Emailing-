# Tabele danych

System używa 8 tabel w n8n (Data Tables). Przed importem workflowów trzeba je utworzyć z dokładnie takimi nazwami kolumn. Kolumny `id`, `createdAt` i `updatedAt` n8n dodaje sam.

## cm_kampanie
Jedna kampania = jedno miasto + jedna branża.

| Kolumna | Typ | Opis |
|---|---|---|
| kampania_id | string | np. `biura-rachunkowe-rzeszow-202611151929` |
| miasto | string | |
| branza | string | |
| warianty | string (JSON) | frazy do Google Maps wymyślone przez AI |
| osm_tagi | string (JSON) | tagi OpenStreetMap, np. `office=accountant` |
| bolaczki_branzy | string (JSON) | typowe problemy branży z rozwiązaniem i korzyścią |
| dni | number | ile dni trwa kampania |
| limit_dzienny | number | ile nowych maili dziennie |
| ocena_ai | number | 1–10 |
| werdykt | string | WARTO / RACZEJ WARTO / NIE WARTO |
| uzasadnienie | string | krótkie podsumowanie oceny |
| ocena_szczegoly | string (JSON) | plusy, minusy, szacunki, alternatywy |
| status | string | `aktywna`, `wstrzymana`, `wyczerpana`, `zakonczona` |
| data_start, data_koniec | date | |
| wariant_idx | number | który wariant frazy jest następny do szukania |
| osm_pobrane | boolean | czy OpenStreetMap był już przeszukany |
| znalezione | number | ile firm znaleziono |
| ostatnie_szukanie | date | |

## cm_firmy
Główna tabela. Jeden wiersz = jedna firma przez cały jej cykl.

| Kolumna | Typ | Opis |
|---|---|---|
| place_id, klucz, zrodlo | string | identyfikacja i źródło (google / osm); `klucz` blokuje duplikaty |
| kampania_id | string | |
| nazwa, miasto, branza, adres, telefon, strona | string | dane z mapy |
| ocena_google, liczba_opinii | number | |
| status | string | patrz niżej |
| powod | string | dlaczego firma odpadła albo czeka |
| email, email_zrodlo | string | adres i miejsce, gdzie został znaleziony |
| email_sprawdzony | string | `ok` / `zly` (weryfikacja DNS) |
| research | string | |
| pomysl_glowny, pomysl_ekstra, typ_propozycji | string | wynik analizy AI |
| priorytet | number | im wyżej, tym wcześniej mail wychodzi |
| temat, tresc | string | gotowy mail |
| ocena_kontrolera | number | ocena kontrolera AI 1–10 |
| proby | number | ile razy research się nie udał |
| etap | number | 0 = nic nie wysłane, 1 = pierwszy mail, 2 = follow-up 1, 3 = koniec |
| data_wyslania, data_nastepnego | date | |
| thread_id, message_id | string | wątek w Gmailu |
| kategoria_odpowiedzi | string | |
| zablokowano | date | blokada na czas researchu |

### Statusy firmy

```
nowa ─► w_researchu ─► gotowa ─► wyslana ─► zakonczona
              │           │          │
              │           │          ├─► zainteresowany
              │           │          ├─► nie_teraz ─► (powrót po kilku tygodniach)
              │           │          └─► odmowa
              │           └─► zly_mail      (weryfikacja DNS)
              ├─► sprawdz_pozniej           (chwilowy błąd, spróbuje ponownie)
              └─► odrzucona                 (brak prawdziwego maila, nie pasuje, słaby mail)
```

## cm_research
| Kolumna | Typ | Opis |
|---|---|---|
| firma_id | number | |
| dane | string (JSON) | zebrany research, używany później do mini-analizy dla zainteresowanych |

## cm_wysylki
Historia każdej wysłanej wiadomości.

| Kolumna | Typ |
|---|---|
| firma_id | number |
| kampania_id, email, typ, temat, tresc, thread_id, message_id, status | string |
| data | date |
| tryb_testowy | boolean |

`typ`: `pierwszy`, `followup1`, `followup2`, `powrot`. `status`: `wyslany`, `odbity`.

## cm_odpowiedzi
| Kolumna | Typ |
|---|---|
| message_id, thread_id, nadawca, kategoria, streszczenie | string |
| firma_id | number |
| data | date |

## cm_czarna_lista
| Kolumna | Typ | Opis |
|---|---|---|
| wartosc | string | adres e-mail albo cała domena |
| typ | string | `email` / `domena` |
| powod | string | |
| data | date | |

## cm_oceny
Ocena rynku zlecana z panelu www (działa w tle, strona odpytuje o wynik).

| Kolumna | Typ |
|---|---|
| ocena_id, status, miasto, branza, wynik | string |
| dni, maile | number |
| data | date |

## cm_ustawienia
Proste pary klucz–wartość.

| klucz | Przykładowa wartość | Opis |
|---|---|---|
| system_wstrzymany | `nie` | `tak` zatrzymuje cały system |
| tryb_testowy | `tak` | `tak` = maile idą tylko do mnie |
| limit_dzienny | `30` | domyślny limit, gdy kampania nie ma swojego |
| wnioski_stylu | (pusty) | wnioski z cotygodniowej analizy, dopisywane do instrukcji pisania |
| firecrawl_alert_data | (pusty) | kiedy ostatnio było ostrzeżenie o kredytach Firecrawla |
| bufor_dni | `2` | na ile dni wysyłki do przodu przygotowywać maile |
