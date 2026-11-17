# 02. Instalacja

Instrukcja dla n8n Cloud. Na n8n self-hosted wygląda to tak samo, trzeba tylko mieć włączone Data Tables.

## 1. Konta i klucze

| Usługa | Do czego | Typ poświadczenia w n8n |
|---|---|---|
| OpenAI | pisanie, analiza, kontrola maili | OpenAI API |
| Google Places API | szukanie firm i opinie | Header Auth (`X-Goog-Api-Key`) |
| Gmail | wysyłka i czytanie odpowiedzi | Gmail OAuth2 |
| Telegram | alerty, raporty, wyłącznik | Telegram API |
| Firecrawl | zapasowe czytanie stron | Firecrawl API |

## 2. Tabele

Utwórz w n8n 8 tabel opisanych w [database/tabele.md](../database/tabele.md). Do `cm_ustawienia` dodaj wiersze z kluczami z tabeli na dole tamtego pliku. Na start ustaw `tryb_testowy = tak`.

## 3. Import workflowów

Zaimportuj pliki z folderu `workflows/` w kolejności numerów (menu **Import from File**). Po imporcie w każdym workflowie:

1. **Tabele.** W każdym węźle tabeli wybierz właściwą tabelę z listy. W plikach w tym miejscu stoi `ID_TABELI_nazwa`, więc od razu widać, o którą chodzi.
2. **Poświadczenia.** Podepnij konta z punktu 1 w węzłach HTTP, Gmail i Telegram.
3. **Wartości do podmiany** (wyszukaj w kodzie):

| Znacznik | Na co zamienić |
|---|---|
| `TWOJA-INSTANCJA.app.n8n.cloud` | adres Twojej instancji n8n |
| `TWOJ_KLUCZ_STEROWANIA` | dowolny długi, losowy ciąg (ten sam we wszystkich workflowach) |
| `TWOJ_KLUCZ_PANELU` | drugi losowy ciąg, hasło do panelu www |
| `TWOJ_TELEGRAM_CHAT_ID` | id czatu, na który mają przychodzić alerty |

Adres nadawcy, podpis i numer telefonu w mailach są wpisane w kodzie węzłów („Gotowy mail”, „Przygotuj pierwszy mail”, „Sprawdź i złóż kolejną wiadomość”). Trzeba je zmienić na swoje.

## 4. Panel www

1. W `panel-www/index.html` podmień `TWOJA-INSTANCJA.app.n8n.cloud` na swój adres.
2. Wrzuć folder `panel-www` na Netlify (Add new project → Deploy manually, przeciągnij folder).
3. Po otwarciu strony wpisz klucz z `TWOJ_KLUCZ_PANELU`.

## 5. Pierwsze uruchomienie

1. Opublikuj (aktywuj) wszystkie 7 workflowów.
2. Utwórz kampanię z panelu albo z formularza „Nowa kampania”.
3. Zostaw tryb testowy na dzień lub dwa. Wszystkie maile przyjdą wtedy na Twoją skrzynkę z dopiskiem `[TEST → adres firmy]`, więc łatwo je przejrzeć.
4. Gdy wszystko wygląda dobrze, wyłącz tryb testowy w panelu.

## Ważne

- Nowa skrzynka Gmail nie powinna od pierwszego dnia wysyłać 30 maili. Lepiej zacząć od 5–10 dziennie i zwiększać limit co kilka dni.
- System sam się zatrzyma, gdy zobaczy za dużo zwrotów. Wtedy trzeba sprawdzić, skąd się wzięły, zanim się go włączy z powrotem.
