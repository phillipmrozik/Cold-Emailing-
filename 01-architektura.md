# 01. Architektura

System składa się z 7 workflowów n8n. Nie wołają się nawzajem. Komunikują się przez wspólne tabele danych, więc każdy workflow można wyłączyć, poprawić albo przetestować osobno, bez psucia reszty.

## Workflowy

### 01. Panel kampanii
Dwa formularze n8n.

- **Nowa kampania.** Miasto, branża, liczba dni i maili dziennie. System bierze do 60 firm z Google Maps, liczy, ile ma własną stronę, a ile tylko profil na platformie (Booksy, Facebook itp.), pobiera kilka stron i szuka na nich maili. Na tej podstawie AI wystawia werdykt: WARTO / RACZEJ WARTO / NIE WARTO, ocenę 1–10, plusy, minusy i bolączki branży, które da się rozwiązać automatyzacją. Kampania startuje dopiero po moim kliknięciu.
- **Panel kampanii.** Podgląd stanu i sterowanie: wstrzymanie lub zakończenie kampanii, wstrzymanie całego systemu, tryb testowy.

### 02. Silnik researchu
Najważniejsza część. Działa po zakończeniu wysyłki (pn–pt od 15:00) oraz w nocy i w weekendy. Przy każdym przebiegu najpierw decyduje, co jest teraz potrzebne:

1. **Szukanie firm**, jeśli w kolejce zostało mało firm ze stroną. Google Maps (kolejne warianty frazy z oceny AI), a na końcu OpenStreetMap. Filtr odrzuca platformy rezerwacyjne i katalogi, a duplikaty wypadają po place_id i domenie.
2. **Research jednej firmy**:
   - pobranie strony głównej i podstron (kontakt, o nas, oferta); gdy strona blokuje pobieranie, wchodzi Firecrawl,
   - opinie z Google,
   - wyszukiwanie w internecie tylko wtedy, gdy maila nie ma na stronie,
   - wybór najlepszego adresu, sprawdzenie źródła, serwera poczty (MX), czarnej listy i tego, czy ten adres już kiedyś dostał maila.
3. **Analiza i mail**:
   - AI ocenia, czy firma pasuje, i wybiera główny temat,
   - drugi model pisze maila według stałego układu,
   - kontrola w kodzie i kontrola przez AI,
   - jedna próba poprawki; jeśli dalej słabo, firma jest odrzucana z opisem powodu.

Każde wywołanie AI idzie najpierw w tańszym trybie `flex`, a gdy ten nie odpowie, automatycznie powtarza się w zwykłym trybie.

### 03. Weryfikacja adresów
Codziennie o 5:30 i co godzinę w czasie wysyłki. Bierze maile gotowe do wysłania, grupuje je po domenie i sprawdza przez DNS:

- czy domena istnieje,
- czy ma serwer poczty (MX), a jeśli nie, to czy ma chociaż adres (A),
- czy nie ma „pustego” MX, czyli domeny, która jawnie nie przyjmuje poczty.

Wysyłka bierze wyłącznie adresy oznaczone jako sprawdzone.

### 04. Wysyłka i follow-upy
Co 30 minut w dni robocze od 6:00 do 15:00. W jednym przebiegu wysyła do 4 wiadomości, z przerwą 1–3 minuty między nimi.

- pomija święta i długie weekendy (API date.nager.at),
- tempo jest rozłożone na cały dzień, więc limit kampanii nie wyjdzie w ciągu pierwszej godziny,
- kolejność: najpierw nowe maile (najlepiej ocenione firmy jako pierwsze), potem follow-upy i powroty po „nie teraz”,
- follow-up pisze AI w tym samym wątku Gmaila, a gdy coś pójdzie nie tak, jest gotowy tekst zapasowy,
- ostatnie sprawdzenie czarnej listy tuż przed wysyłką,
- tryb testowy: wszystko idzie na moją skrzynkę z dopiskiem `[TEST → adres firmy]`.

### 05. Odpowiedzi i raporty
- co 15 minut w dzień i co godzinę w nocy czyta nowe wiadomości ze skrzynki,
- rozpoznaje zwroty (bounce) i autorespondery bez użycia AI, a resztę klasyfikuje AI,
- „nie teraz” wraca do kolejki za kilka tygodni, a odmowa trafia na czarną listę,
- dla zainteresowanych przygotowuje w Gmailu szkic odpowiedzi z krótką analizą firmy i wysyła mi powiadomienie na Telegram,
- pilnuje zwrotów: gdy wśród ostatnich wysyłek jest ich za dużo, sam wstrzymuje cały system i daje znać, żeby nie zepsuć reputacji skrzynki,
- raport dnia o 18:00 i w niedzielę wnioski z tygodnia, które trafiają też do ustawień jako wskazówki do pisania.

### 06. API strony panelu
Webhooki dla panelu www: stan systemu, ocena rynku w tle, start kampanii, sterowanie, lista maili na dziś i odrzucenie maila z kolejki. Każde zapytanie musi mieć tajny klucz. Odpowiedzi są tak zbudowane, żeby strona na Netlify mogła je czytać bez problemów z CORS.

### 07. Awaryjny wyłącznik
Jeden webhook podpięty pod przycisk w alertach na Telegramie. Zapisuje w ustawieniach `system_wstrzymany = tak`, a wszystkie pozostałe workflowy sprawdzają to na samym początku i nic nie robią.

## Obsługa błędów

Każdy workflow ma własną ścieżkę błędów:
- pojedyncze, chwilowe błędy są pomijane (system spróbuje przy następnym przebiegu),
- błąd jest zgłaszany na Telegram dopiero, gdy się powtarza albo jest poważny (wygasłe logowanie, skończone środki, limit Gmaila),
- ten sam problem nie jest zgłaszany częściej niż co 3 godziny,
- każdy alert mówi prostymi słowami, co się stało i co z tym zrobić.

## Dlaczego wbudowane tabele n8n, a nie zewnętrzna baza

Przy tej skali (setki firm na kampanię) tabele n8n w zupełności wystarczają. Nie trzeba utrzymywać osobnej bazy, a każdą tabelę widać i można edytować bezpośrednio w n8n. Przejście na Postgresa lub Supabase jest możliwe, bo cała logika siedzi w workflowach, a nie w bazie.
