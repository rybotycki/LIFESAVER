# Lifesaver — specyfikacja funkcjonalna v1.0

**Status:** wymagania wstępnie zatwierdzone. Zakres E2EE i sposób kontroli dostępu do pozostałych danych zostały następnie zaakceptowane w [ADR-001](adr.md#adr-001). Kwestie do doprecyzowania są zebrane w [liście otwartej](kwestie-otwarte.md).

## 1. Cel

Lifesaver to prywatna PWA dla jednej osoby. Służy do codziennego śledzenia nawyków, zmiany niepożądanych zachowań i obserwacji długoterminowych postępów. Priorytety to prosta obsługa, rzetelne statystyki, dyskrecja i utrzymanie regularności przez wiele miesięcy.

## 2. Platformy i dostęp

- Aplikacja jest dostępna na iPhonie i komputerze. Historia i ustawienia synchronizują się między urządzeniami.
- Użytkownik loguje się kontem Google. Firebase Authentication i reguły Firestore chronią dane poza zakresem E2EE.
- E2EE obejmuje wyłącznie dwa prywatne nawyki z tabeli i prywatny licznik, wraz z danymi, które ujawniają ich treść lub wyniki. Dostęp do tych danych wymaga osobnej blokady i sposobu odzyskiwania.
- Ekrany ogólnodostępne i powiadomienia ukrywają informacje prywatne.
- Preferowane są usługi darmowe, bez obowiązkowego abonamentu. Wybór pozostałych technologii opisuje [propozycja architektury](architektura.md).

## 3. Nawyki codzienne na start

| Nr | Nawyk | Warunek zaliczenia |
| --- | --- | --- |
| 1 | Mycie zębów rano | Wykonano |
| 2 | Mycie zębów wieczorem | Wykonano |
| 3 | Prysznic | Wykonano |
| 4 | Pobudka przed 9:00 | Pobudka przed 09:00 |
| 5 | Spacer | Co najmniej 15 minut |
| 6 | Nauka zawodowa | Co najmniej 1 minuta rzeczywistej nauki |
| 7 | TikTok | Maksymalnie 120 minut dziennie |
| 8 | Bez materiałów seksualnych w nocy | Brak korzystania od 00:00 do zaśnięcia. Nawyk prywatny |
| 9 | Bez generowania materiałów seksualnych | Brak generowania takich materiałów w ciągu dnia. Nawyk prywatny |
| 10 | Zaśnięcie przed 1:00 | Zaśnięcie przed 01:00 |

- Wszystkie początkowe cele można później zmienić. Przy nauce zawodowej aplikacja zapisuje rzeczywisty czas, niezależnie od progu zaliczenia. Zakres obejmuje m.in. SQL, Power BI, ETL i przygotowanie zawodowe w IT, z pominięciem zajęć na studiach. Cel 3–4 godzin dziennie służy jako punkt odniesienia poza oceną nawyku.
- Sen zapisuje godzinę zaśnięcia i pobudki. Noc przypada na dzień pobudki. W v1 wystarczy ręczny zapis. Zasady interpretacji progu 01:00 przy przejściu przez północ wymagają doprecyzowania.
- Czas TikToka można wprowadzać ręcznie. Użytkownik może później zmienić limit.

## 4. Prywatny licznik masturbacji

- Moduł rejestruje dzienną liczbę zdarzeń. Przyciski zwiększają i zmniejszają wartość, a historię można korygować.
- Jawne zero oznacza zarejestrowany wynik. Brak danych to osobny stan.
- Moduł pokazuje statystyki dzienne, tygodniowe i miesięczne, średnią dzienną oraz trend.
- Licznik służy obserwacji zachowań bez zawstydzania, limitów, automatycznej oceny i nagród za abstynencję. Pozostaje poza ogólnym wynikiem nawyków.
- Profesjonalna pomoc przy zachowaniach kompulsywnych pozostaje poza zakresem modułu.

## 5. Ocena nawyków

| Stan | Znaczenie | Wpływ na realizację |
| --- | --- | --- |
| Tak | Warunek spełniony | 100% dla wpisu |
| Nie | Warunek niespełniony | 0% dla wpisu |
| Brak danych | Wpis jeszcze nieoceniony. Stan domyślny | Pomijany w mianowniku realizacji |

Każdy wpis ma jeden stan. Przy nawykach z pomiarem aplikacja ocenia wpis według wartości i warunku nawyku. Pierwszeństwo ręcznej korekty statusu przy istniejącym pomiarze pozostaje otwarte.

## 6. Statystyki i analiza postępów

- Aplikacja pokazuje ogólny procent realizacji codziennych nawyków, realizację każdego z nich oraz osobno kompletność danych.
- Realizacja = liczba wpisów „Tak” ÷ liczba wpisów „Tak” i „Nie” × 100%.
- Kompletność = liczba wpisów „Tak” i „Nie” ÷ liczba wszystkich wymaganych wpisów × 100%.
- Przykład z fikcyjnymi danymi. 6 wpisów „Tak”, 2 „Nie” i 2 bez danych daje 75% realizacji oraz 80% kompletności. Przy braku ocen realizacja jest niedostępna. Gdy okres nie zawiera żadnych wymaganych wpisów, kompletność też nie ma podstawy do obliczenia.
- Wszystkie codzienne nawyki mają równą wagę. Pełny wynik uwzględnia oba prywatne nawyki z zachowaniem osobnej blokady. Sposób prezentacji wyniku w widoku ogólnym pozostaje do ustalenia.
- Dostępne są wyniki tygodniowe, miesięczne i długoterminowe, porównania okresów i wykresy trendów. Każdy nawyk ma własny ekran z historią, wykresami i odpowiednimi pomiarami.
- Filtrowanie według kategorii jest opcjonalne. Forma kalendarza historii i szczegóły wykresów pozostają otwarte.

## 7. Nawigacja i historia

- Szybkie przejście do dzisiaj i wczoraj oraz dostęp do wszystkich wcześniejszych dni.
- Edycja statusów, pomiarów, godzin i licznika w całej historii z automatycznym przeliczeniem statystyk.
- Zaliczenie można zapisać wyłącznie dla bieżącej lub przeszłej daty aktywności.
- Data aktywności jest odrębna od daty wprowadzenia i ostatniej zmiany informacji.

## 8. Cele tygodniowe

Cele codzienne ocenia się osobno każdego dnia. Cele tygodniowe rozlicza się w skali tygodnia według liczby wykonań lub wartości, z własnymi wskaźnikami i statystykami. Początkowa lista celów tygodniowych jest pusta.

## 9. Personalizacja

Użytkownik może dodawać nawyki i cele, zmieniać nazwę, warunek, częstotliwość i wartości docelowe oraz archiwizować i przywracać nawyki z zachowaniem historii. Trwałe usunięcie wymaga potwierdzenia. Obsługiwane są nawyki oceniane bezpośrednio oraz cele mierzone czasem, liczbą wykonań lub inną wartością liczbową.

## 10. Codzienne korzystanie i przypomnienia

- Wpis można zapisać od razu po wykonaniu lub uzupełnić zbiorczo później. Zapis wymaga możliwie małej liczby kroków.
- Konfigurowalne przypomnienie wieczorne pomaga uzupełnić brakujące wpisy. Ma neutralną treść i pozostawia status wpisów bez zmian.
- Dodatkowe przypomnienia dla pojedynczych nawyków pozostają do ustalenia.

## 11. Filozofia produktu

Aplikacja pokazuje sukcesy, niepowodzenia i braki danych, zachowuje wcześniejszy postęp po gorszym dniu i wspiera porównania w czasie. Miarą postępu jest regularność i stopniowa poprawa. Użytkownik sam decyduje o rozkładzie dnia i czasie wolnym.

## 12. Zakres odłożony

Planowanie godzinowe, rozbudowane zarządzanie zadaniami i projektami, obowiązkowa integracja kalendarza, automatyczny odczyt wszystkich danych telefonu, pełne integracje Apple Health i Sleep Cycle, zaawansowana motywacja oraz domyślne cele tygodniowe pozostają poza wymaganym zakresem v1. Integracje snu można osobno ocenić technicznie, również pod kątem danych z Apple Watch.

## 13. Oczekiwany rezultat

Użytkownik może wygodnie i dyskretnie rejestrować dziesięć nawyków oraz prywatny licznik, poprawiać historię, analizować postępy i zmieniać własne cele na iPhonie i komputerze.
