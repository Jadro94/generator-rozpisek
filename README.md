# Rozpiska — generator dokumentacji produkcyjnej

Narzędzie zamienia drabinkę odcinka serialu w gotową rozpiskę produkcyjną:
lista castingowa, pozostali odtwórcy, statyści, lokacje (stałe i płatne), PPS,
pojazdy i podsumowanie — w formacie zgodnym ze wzorcem produkcyjnym.

## Jak to działa

Aplikacja dzieli pracę na dwie części:

**Deterministycznie (parser)** — wszystko, co da się policzyć z nagłówków scen:
numery scen, przypisania postaci do scen, offy, lokacje i sublokacje, liczba scen
dzień/noc, wykrycie PPS-ów, podsumowanie liczbowe. Tu nie ma miejsca na pomyłkę
modelu — numeracja zawsze się zgadza.

**Przez AI (opcjonalnie)** — tylko to, czego policzyć się nie da: opisy postaci,
wiek tam gdzie nie wynika z tekstu, treść materiałów PPS, pojazdy i notatki do
lokacji. Wszystko można potem ręcznie poprawić.

## Uruchomienie

To jeden plik HTML bez zależności i bez backendu.

- **Lokalnie:** otwórz `index.html` dwuklikiem w przeglądarce. Wczytywanie
  plików Word, wszystkie wyliczenia i eksport do `.docx` działają bez internetu.
- **Dla zespołu:** wrzuć `index.html` na dowolny hosting statyczny — Netlify,
  Cloudflare Pages, GitHub Pages, zwykły serwer WWW. Nie ma kroku budowania,
  więc wystarczy sam plik.

Jedyna funkcja, która wymaga połączenia z siecią, to przycisk „Uzupełnij przez
API”. Tryb „Kopiuj prompt” działa zawsze — również przy pracy offline.

## Rozpoznawany format drabinki

```
12. WN. KOMENDA - POKÓJ PRZESŁUCHAŃ. DZIEŃ 2 (14.30)
(Adam, Olga, Woźniak, Krysia Górska, Żaneta-off)
Treść sceny…
```

Parser radzi sobie z wariantami spotykanymi w praktyce:

- `WN` / `PL` / `WNĘTRZE` / `PLENER`, z kropką lub bez, ze spacją lub bez
- separator lokacja–sublokacja: `-`, `–`, `—`
- pora dnia: `DZIEŃ`, `NOC`, `ŚWIT`, `WIECZÓR`, `RANEK`, `POŁUDNIE`, `ZMIERZCH` (+ numer dnia)
- adnotacje w nawiasie: `(KONTYNUACJA, 16.06)`, `(CHWILĘ PÓŹNIEJ, 14.59)`, `(NASTĘPNEGO DNIA, 7.34)`
- nagłówki zawinięte na dwie linie
- numery scen z literą: `17`, `17A`, `17B`
- prefiks `SC.` przed numerem
- offy: `Żaneta-off`, `Olga (off)`, `Radosław Więcek – off`
- liczebność statystów: `Mundurowy x4`, `Mundurowi x 2 z radiowozem`

Jeśli w nagłówkach w ogóle nie ma numerów scen, aplikacja ponumeruje je kolejno
i o tym uprzedzi.

## Wczytywanie plików

Przeciągnij plik na pole wczytywania albo kliknij, żeby go wybrać.

| Format | Obsługa |
|---|---|
| `.docx` | tak — dokument jest rozpakowywany i czytany lokalnie w przeglądarce |
| `.txt` | tak |
| `.rtf` | tak (podstawowa ekstrakcja tekstu) |
| `.fdx` | tak (Final Draft) |
| `.doc` | nie — otwórz w Wordzie i zapisz jako `.docx` |

Odczyt Worda przetestowano na plikach z Worda, z LibreOffice i z eksportu
Google Docs. Radzi sobie z drabinką w tabeli, z akapitami porozbijanymi na
wiele fragmentów po edycjach oraz z dokumentami ze śledzonymi zmianami —
tekst usunięty w recenzji jest pomijany, tak jak w widoku „Końcowy”.

**Automatyczna numeracja Worda.** Jeśli numery scen pochodzą z listy
numerowanej Worda, a nie są wpisane w treść, w tekście dokumentu ich nie ma —
Word trzyma je osobno. Aplikacja to wykryje, ponumeruje sceny kolejno od 1
i wyraźnie o tym uprzedzi. Gdy numeracja miałaby się nie zgadzać (bo np.
odcinek zawiera scenę `17A` albo numeracja nie zaczyna się od 1), zaznacz
w Wordzie całość, skopiuj i wklej tekst wprost do aplikacji — wtedy numery
przeniosą się jako zwykły tekst.

## Konfiguracja pod własny serial

Zakładka **Ustawienia** pozwala dostosować narzędzie bez ruszania kodu:

- **Postacie stałe** — lista postaci trafiających do „Listy castingowej”, wraz
  z aliasami (`Janusz Woźniak | Woźniak, Janusz, kom. Woźniak, Komendant`).
  Parser dopasowuje też formy ze stopniem służbowym.
- **Słowa-klucze statystów** — nazwy zawierające któreś z tych słów lądują
  domyślnie w statystach.
- **Lokacje stałe** — nazwy lokacji niepłatnych (komenda, radiowozy).

Każdą pozycję można też przenieść ręcznie przyciskami `→ epizody` / `→ statyści`
/ `→ castingowa` w zakładce Rozpiska.

## AI — dwa tryby

1. **Z kluczem API** — wpisz klucz Anthropic w Ustawieniach i kliknij
   „Uzupełnij przez API”. Klucz żyje tylko w pamięci karty przeglądarki,
   nie jest nigdzie zapisywany ani wysyłany poza `api.anthropic.com`.
   Przycisk „Pobierz listę” pobiera aktualne identyfikatory modeli z API.
2. **Bez klucza** — „Kopiuj prompt”, wklej go w Claude albo ChatGPT,
   a zwrócony JSON wklej z powrotem przyciskiem „Wklej odpowiedź AI”.

Drugi tryb nie wymaga niczego poza przeglądarką — to sensowny domyślny wybór
dla osób, które nie mają konta deweloperskiego.

## Eksport

- **Kopiuj do schowka** — sformatowany tekst z punktorami, gotowy do wklejenia
- **.docx** — dokument Word budowany w przeglądarce, bez żadnych bibliotek
- **.txt** — czysty tekst
- **Projekt .json** — cała praca (drabinka + ręczne poprawki + ustawienia)
  do zapisania i przekazania dalej

## Publikacja online

Narzędzie to jeden plik statyczny bez kroku budowania, więc zadziała na każdym
hostingu. Wybór zależy od tego, kto ma mieć dostęp.

**Tylko ekipa, bez zakładania czegokolwiek** — wyślij plik `index.html` mailem.
Działa po dwukliku, łącznie z wczytywaniem Worda i eksportem. Żadna drabinka
nie opuszcza wtedy komputera.

**Jeden zawsze aktualny adres, najszybciej** — [Netlify Drop](https://app.netlify.com/drop):
wrzuć folder z plikiem metodą przeciągnij-i-upuść, dostajesz adres `*.netlify.app`.
Konto potrzebne dopiero, żeby zatrzymać stronę na stałe. Uwaga: na kontach
założonych po wrześniu 2025 ochrona hasłem jest funkcją planu Pro — na darmowym
planie „prywatność” sprowadza się do nieudostępniania adresu.

**Realna kontrola dostępu za darmo** — Cloudflare Pages + Cloudflare Access
z logowaniem kodem na e-mail. Plan Zero Trust obejmuje do 50 użytkowników.
Polityka Access włączana jednym przełącznikiem chroni *wdrożenia podglądowe*;
objęcie nią adresu produkcyjnego wymaga dodatkowej konfiguracji.

**Publicznie, na dłużej** — GitHub Pages: darmowe dla repozytoriów publicznych,
HTTPS i własna domena w cenie, limit 1 GB rozmiaru i 100 GB transferu
miesięcznie. Pages na repozytorium prywatnym wymaga planu Pro.

Koszty użycia AI ponosi każdy użytkownik osobno — klucz API wpisuje własny,
więc opublikowanie narzędzia nie generuje rachunków po stronie wydawcy.

## Streszczenia odcinków

Zakładka 4 generuje streszczenie w standardzie prasowo-zapowiedziowym:
nagłówek `Odc. 59 (1274) – TYTUŁ (para Adam/Olga)`, akapit sprawy kryminalnej
zakończony pytaniem budującym napięcie i akapit wątku prywatnego, 80–150 słów.
