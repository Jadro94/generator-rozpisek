# Rozpiska — generator dokumentacji produkcyjnej

### ▶ [Uruchom narzędzie](https://jadro94.github.io/generator-rozpisek/)

`https://jadro94.github.io/generator-rozpisek/`

Nic nie trzeba pobierać ani instalować — narzędzie otwiera się w przeglądarce.
Kliknięcie pliku `index.html` na liście powyżej pokaże tylko kod źródłowy;
działającą aplikację uruchamia dopiero powyższy adres.

---

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

## Strona startowa

Aplikacja otwiera się ekranem powitalnym z logo, krótkim opisem i pięcioma krokami
„Jak to działa?". Przycisk **START** przechodzi do zakładki „Drabinka" i ustawia
kursor w polu tekstowym. Kliknięcie logo w nagłówku wraca na stronę startową.

Logo jest osadzone w pliku HTML, więc narzędzie nadal jest jednym plikiem bez
zewnętrznych zasobów i działa bez internetu.

## Dwa źródła: drabinka albo scenariusz

Po przycisku START aplikacja pyta, z czego robimy rozpiskę. Efekt jest ten sam —
różni się tylko zapis obsady pod nagłówkiem sceny. Wybór można zmienić w każdej
chwili odnośnikiem „zmień źródło" w zakładce 1.

### Scenariusz

```
4) PL. PRZED DOMEM SUDECKIEJ/ULICA        DZIEŃ 1 (19:43)
O: Adam (T), Olga (T), Mundurowy 1, Mundurowy 2 (OFF)
R: Worek na zwłoki, żółta taśma policyjna
M: Sudecka (otarcia na szyi)
K: Jan (kombinezon technika)
P: Nieoznakowany radiowóz, Radiowóz
```

- **Obsada** pochodzi z wiersza `O:`. Jeśli scena ma kilka takich wierszy
  (drugi blok opisuje np. to, co widać na materiale PPS), brane są wszystkie —
  te postacie też trzeba obsadzić.
- **Pojazdy** pochodzą z wierszy `P:` i trafiają wprost do listy pojazdów,
  z numerami scen.
- **Znacznik `(T)`** oznacza postać z kwestiami. Postać, która nie ma go
  w żadnej scenie, trafia wstępnie do statystów — chyba że jej imię zachowuje się
  w tekście jak nazwa własna (zamordowana ofiara widziana tylko na nagraniu,
  „Mundurowy Edward”). To wciąż tylko podpowiedź: **przejrzyj podział w zakładce 2**
  i przenieś, co trzeba, jednym kliknięciem.
- **Tytuły PPS** aplikacja wyłapuje z wierszy `R:` — zapis `PPS2 – nagranie
  schadzki` sam uzupełnia nazwę materiału.
- **PPS jako osobna scena.** Gdy wstawka ma własny wiersz `O:`, jest to materiał
  do nakręcenia osobno. Grające w nim postacie dostają w liście scen pozycję
  `PPS1`, `PPS2` zamiast numeru sceny — liczy się ona do sumy scen aktora
  i stoi na końcu listy, za numerami zwykłych scen:

  ```
  2. Tatiana (29) – 3 (2, PPS1, PPS2)
  ```

  Numer PPS-u rozpoznajemy po najbliższej wzmiance nad wstawką („na ekranie
  widać PPS1”). Materiał odtwarzany w kilku scenach figuruje raz, bo kręci się
  go raz.
- Wiersze `R:`, `K:`, `M:` nie są poza tym wykorzystywane.

Obsługiwane są też drobne literówki spotykane w praktyce: brak przecinka po
`(T)` („Roksana (T) Mundurowy 1”), sklejona kropka w nazwie lokacji
(`MIESZKANIE MIROSŁAWY.SALON`) oraz nagłówki pisane raz wersalikami, raz
mieszanką — nazwy lokacji są ujednolicane, więc `DOM BRODZKICH` i
`Dom brodzkich` to jedna pozycja.

## Rozpoznawany format drabinki

```
12. WN. KOMENDA - POKÓJ PRZESŁUCHAŃ. DZIEŃ 2 (14.30)
(Adam, Olga, Woźniak, Krysia Górska, Żaneta-off)
Treść sceny…
```

Parser radzi sobie z wariantami spotykanymi w praktyce:

- `WN` / `PL` / `WNĘTRZE` / `PLENER`, z kropką lub bez, ze spacją lub bez
- separator lokacja–sublokacja: `-`, `–`, `—`, a gdy myślnika nie ma — ukośnik
  (`NIEOZNAKOWANY RADIOWÓZ/BOCZNA ULICZKA`). Ukośnik wewnątrz samej sublokacji
  (`PARK - ALEJKA/PARKING`) zostaje nietknięty.
- pora dnia: `DZIEŃ`, `NOC`, `ŚWIT`, `WIECZÓR`, `RANEK`, `POŁUDNIE`, `ZMIERZCH` (+ numer dnia)
- adnotacje w nawiasie: `(KONTYNUACJA, 16.06)`, `(CHWILĘ PÓŹNIEJ, 14.59)`, `(NASTĘPNEGO DNIA, 7.34)`
- nagłówki zawinięte na dwie linie
- numery scen z literą: `17`, `17A`, `17B`, `17AB`, a także `17 A`, `17-A`, `17/A`
- numer zakończony kropką, nawiasem, dwukropkiem albo samą spacją: `17.` `17)` `17:` `17 `
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
i wyraźnie o tym uprzedzi.

**Dokument mieszany — numeracja Worda plus scena `8A` dopisana ręcznie.**
To częsty układ: scenariusz numeruje lista Worda, ale scenę dostawioną między
8 a 9 trzeba wpisać ręcznie jako `8A.`, bo lista tego nie potrafi. Aplikacja
czyta wtedy oba rodzaje nagłówków naraz: numery wpisane w tekst zachowuje bez
zmian, a brakujące uzupełnia po kolei, licząc od ostatniego znanego — dzięki
czemu `8A` nie zabiera numeru 9 kolejnej scenie. Pojawia się przy tym
ostrzeżenie, w ilu nagłówkach numer faktycznie był.

Gdy numeracja miałaby się nie zgadzać (bo np. nie zaczyna się od 1),
zaznacz w Wordzie całość, skopiuj i wklej tekst wprost do aplikacji — wtedy
numery przeniosą się jako zwykły tekst i nic nie trzeba odtwarzać.

## Sklejanie wariantów tej samej lokacji

Radiowóz bywa zapisywany na kilka sposobów w jednym odcinku:
`NIEOZNAKOWANY RADIOWÓZ/ULICA (W RUCHU)`, `NIEOZNAKOWANY RADIOWÓZ/BOCZNA ULICZKA`,
`NIEOZNAKOWANY RADIOWÓZ W RUCHU`, `NIEOZNAKOWANY RADIOWÓZ - PARKING`. Dla produkcji
to jedna lokacja, więc aplikacja skleja je w jedną pozycję, a to, co stoi po nazwie
kanonicznej, traktuje jako sublokację:

```
1. Nieoznakowany radiowóz – 3 sceny dzień, 1 scena noc
   ○ Ulica (w ruchu) (4, 12)
   ○ Boczna uliczka (9)
   ○ Parking (16)
```

Nazwy kanoniczne bierze z listy **Lokacje stałe** w Ustawieniach — dopisanie tam
własnej nazwy (np. `karetka`) sprawi, że jej warianty też będą się sklejać.
Dopasowanie idzie od najdłuższej nazwy, więc `nieoznakowany radiowóz` wygrywa
z samym `radiowóz`.

## Co można poprawić w zakładce 2

Zakładka „Rozpiska" to miejsce na korektę tego, co aplikacja odczytała.
Każda zmiana natychmiast przelicza całą rozpiskę.

- **Numery scen** — osobna karta na górze zakładki. Numer każdej sceny można
  nadpisać, także na numer z literą (`8A`). Obok numeru widać lokację, porę dnia
  i obsadę sceny, więc wiadomo, co się poprawia. Sceny ponumerowane automatycznie
  są oznaczone. Przycisk **Przenumeruj kolejno od 1** nadaje numery od nowa,
  **Przywróć numery z drabinki** cofa wszystkie ręczne zmiany. Powtórzony numer
  jest podświetlany na czerwono wraz z ostrzeżeniem.
- **Przynależność postaci** — przyciski `→ epizody` / `→ statyści` / `→ castingowa`
  przenoszą postać między sekcjami; strzałki ↑↓ ustawiają kolejność postaci stałych.
- **Nazwy, wiek i opisy** — wszystko edytowalne wprost w polach.
- **Lokacje** — nazwa, notatka realizacyjna, przełącznik `stała` / płatna,
  nazwy sublokacji (identyczne nazwy scalają się w jedną pozycję).
- **Listy scen** — przy każdej postaci, lokacji i sublokacji jest pole `Sceny`,
  które można nadpisać ręcznie. Numery po przecinku; przy postaci można dopisać
  `(off)`. Zmiana listy przy lokacji przelicza od nowa podział dzień/noc.
- **Usuwanie pozycji** — przycisk `usuń` przy postaci, lokacji, sublokacji i PPS-ie.
  Usunięcie jest trwałe: pozycja nie wraca przy przenumerowaniu scen, zmianie
  ustawień ani po wczytaniu zapisanego projektu.
- **PPS i pojazdy** — nazwa, dzień, opis; pojazdy dodaje się i usuwa ręcznie.

Ręcznie poprawione numery scen zapisują się w pliku projektu razem z resztą zmian.

## Nawigacja

U dołu każdej zakładki są przyciski **← Wstecz** i **Dalej →** prowadzące przez
kolejne kroki: drabinka → rozpiska → podgląd i eksport → streszczenie → ustawienia.

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

## Format gotowej rozpiski

- **Lista castingowa, Pozostali odtwórcy, Statyści** — listy numerowane,
  każda liczona od 1.
- **Lokacje** — numerowane, osobno w sekcji „Lokacje stałe" i „Lokacje".
- **Sublokacje** — punktory `○`, wcięte pod swoją lokacją.
- **Numery scen** podawane są przy sublokacjach, a nie przy lokacji, żeby nie
  dublować tej samej informacji dwa razy. Wyjątek: lokacja bez sublokacji
  (wtedy numery stoją przy niej) oraz sceny, które do żadnej sublokacji nie
  należą — te też zostają przy lokacji, bo inaczej zniknęłyby bez śladu.
- **PPS, pojazdy i podsumowanie** — bez zmian, punktory `●`.

## Eksport

- **Kopiuj do schowka** — sformatowany tekst z punktorami, gotowy do wklejenia
- **.docx** — dokument Word budowany w przeglądarce, bez żadnych bibliotek; podgląd
  na ekranie i gotowy dokument są w tym samym kroju pisma (Arial)
- **.txt** — czysty tekst
- **Projekt .json** — cała praca (drabinka + ręczne poprawki + ustawienia)
  do zapisania i przekazania dalej

## Prywatność i poufność materiałów

Drabinki i scenariusze przed emisją są materiałem poufnym, więc poniżej dokładnie,
co się z nimi dzieje. Wszystkie poniższe stwierdzenia zostały zweryfikowane
pomiarem ruchu sieciowego w przeglądarce, nie tylko przeglądem kodu.

### Co zostaje na Twoim komputerze

Aplikacja to jeden plik bez backendu. Wczytanie pliku Word, rozpakowanie go,
parsowanie nagłówków, wszystkie wyliczenia, edycja i eksport do `.docx` dzieją
się w całości w przeglądarce.

Zmierzony ruch sieciowy przy pełnym obiegu pracy — wczytanie `.docx`, edycja
opisów, eksport `.docx`, eksport `.txt`, zapis projektu — to **jedno zapytanie:
pobranie samej strony**. Żadna treść drabinki nie jest nigdzie wysyłana.

Aplikacja nie ładuje też niczego z zewnątrz: zero czcionek z Google, zero
bibliotek z CDN, zero analityki, zero skryptów śledzących. Po całej sesji pracy
w przeglądarce zostaje: **0 wpisów w localStorage, 0 w sessionStorage,
0 ciasteczek, 0 baz IndexedDB**. Nic nie zostaje zapisane.

> Adresy `schemas.openxmlformats.org` widoczne w kodzie **nie są** pobieraniem
> czegokolwiek — to identyfikatory przestrzeni nazw wymagane wewnątrz formatu
> `.docx`. Nigdy nie następuje pod nie żadne połączenie.

### Kiedy materiał opuszcza komputer

Dokładnie w jednym momencie: gdy sam klikniesz **„Uzupełnij przez API"** albo
**„Kopiuj prompt"**. Oba przekazują modelowi **pełną treść drabinki**, razem
z zakończeniem sprawy — nie streszczenie, nie same nagłówki.

Przycisk **„Pokaż, co zostanie wysłane"** wyświetla dokładną treść przed
wysłaniem. Warto go użyć choć raz, żeby zobaczyć skalę.

Dwie drogi różnią się zasadami:

| Droga | Zasady |
|---|---|
| **Własny klucz API** (przycisk „Uzupełnij przez API") | Anthropic nie trenuje modeli na danych z API bez wyraźnej zgody, a treść zapytań domyślnie nie jest przechowywana. Organizacje mogą dodatkowo poprosić o tryb zerowej retencji (ZDR). |
| **Kopiuj prompt → wklejenie w Claude / ChatGPT** | Obowiązują zasady konta konsumenckiego danego dostawcy, nie warunki API. W Claude uczenie na rozmowach jest wyborem użytkownika w ustawieniach — sprawdź, jak masz to ustawione. W innych narzędziach reguły bywają inne. |

Jeśli materiał jest objęty NDA, bezpieczniejsza jest droga przez własny klucz API.

### Czego narzędzie nie chroni

Uczciwie o granicach:

- **Plik projektu `.json`** zawiera pełną drabinkę zwykłym tekstem, bez
  szyfrowania. Traktuj go jak sam scenariusz.
- **Wyeksportowane `.docx` / `.txt`** lądują w folderze pobierania — istotne
  na komputerach współdzielonych.
- **Schowek.** „Kopiuj prompt" umieszcza całą drabinkę w schowku systemowym.
  Menedżery schowka i synchronizacja schowka między urządzeniami mogą to przechwycić.
- **Rozszerzenia przeglądarki** z dostępem do treści stron widzą zawartość
  każdej otwartej karty — dotyczy to dowolnej aplikacji webowej, także tej.
- **Podmieniona kopia.** Ktoś może postawić własną, zmodyfikowaną wersję tego
  pliku pod innym adresem. Rozsyłaj ekipie wyłącznie oficjalny adres.

### Zalecenie dla zespołu

Do pracy na materiale przed emisją: używaj trybu offline (bez klikania przycisków
AI) albo własnego klucza API. Opisy postaci, których nie uzupełni model, i tak
zwykle szybciej wpisuje się ręcznie, niż poprawia po modelu — a numeracja,
lokacje i zliczenia, czyli najbardziej pracochłonna część, liczą się bez udziału AI.

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
