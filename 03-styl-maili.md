# 03. Styl maili

Najwięcej czasu zajęło mi nie technika, tylko to, żeby maile nie brzmiały jak bot albo spam. Pierwsze wersje były „personalizowane” na siłę („zauważyłem na Państwa stronie…”, „pewnie co miesiąc…”) i właśnie przez to wyglądały sztucznie. Teraz zasada jest odwrotna: zero zgadywania i bardzo prosty język.

## Układ

Każdy mail ma te same części, zawsze w tej kolejności:

1. **Powitanie.** Dzień dobry / Witam.
2. **Kim jestem.** Jedno zdanie: imię, nazwisko i że zajmuję się automatyzacją AI.
3. **Co to jest.** 3–4 krótkie zdania tak, żeby zrozumiał każdy: w każdej firmie ktoś codziennie robi ręcznie te same rzeczy przy komputerze, automatyzacja sprawia, że dzieją się same, szybko i bez błędów.
4. **Co firma z tego ma.** Zawsze trzy rzeczy wprost: więcej klientów, większe zyski i przewaga nad konkurencją. Do tego powód: nie tracą czasu na ręczną pracę, tylko skupiają się na klientach i rozwoju.
5. **Trzy przykłady.** Pierwszy typowy dla tej branży, dwa pozostałe takie, które ma każda firma (faktury, dokumenty, przypomnienia, zapytania od klientów).
6. **Cena i pytanie.** Uczciwie: buduję portfolio, więc jest taniej. Na koniec propozycja darmowej rozpiski i pełne pytanie, czy mogę ją przesłać.

Podpis z numerem telefonu i adresem strony dodaje kod, a nie AI, więc zawsze jest taki sam.

## Czego mail nie może zawierać

Kontrola w kodzie odrzuca mail, gdy znajdzie:

- zgadywanie: *pewnie, chyba, myślę, wydaje mi się, być może, zapewne*,
- powoływanie się na stronę lub opinie firmy: *zauważyłem, przeczytałem, na Państwa stronie*,
- formułki: *w praktyce, oznacza to, przy okazji, dzięki czemu, krótko się przedstawię*,
- słowa techniczne: *workflow, integracja, API, n8n, CRM, chatbot, system, algorytm*,
- propozycję strony internetowej, SEO albo reklam (robię automatyzacje, nie strony),
- wykrzykniki, emotikony, „Szanowni Państwo”, procenty i wymyślone liczby,
- brak którejś z trzech korzyści: klientów, zysków albo przewagi nad konkurencją,
- pytanie z jednego słowa na końcu („Przesłać?”),
- tekst zbyt podobny do innych maili w kolejce (porównanie po trójkach słów).

Do tego automatycznie poprawiam częsty błąd odmiany („daje przewaga” → „daje przewagę”).

## Kontroler AI

Po kontroli w kodzie mail czyta drugi model AI i ocenia go od 1 do 10 według 10 punktów: prosty język, brak błędów, czy brzmi jak człowiek, brak zgadywania, wyjaśnienie automatyzacji, korzyści, trzy punkty, cena, pytanie końcowe i brak propozycji strony. Poniżej progu mail idzie do jednej poprawki z konkretnymi wskazówkami. Jeśli dalej nie przejdzie, firma jest odrzucana.

## Przykład

Gotowy mail z systemu jest w [examples/przyklad-maila.md](../examples/przyklad-maila.md).
