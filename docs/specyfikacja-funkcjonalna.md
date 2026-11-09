# Lifesaver — specyfikacja funkcjonalna v1.0

**Stan na 9 listopada 2026.** Specyfikacja opisuje docelowy zakres prywatnej PWA oraz aktualne zachowanie MVP. W środowisku testowym działa osiem zwykłych nawyków, Historia i personalizacja. Strefa prywatna, cele tygodniowe i przypomnienia pozostają do wdrożenia. [ADR](adr.md) · [Kwestie otwarte](kwestie-otwarte.md)

## 1. Cel

Lifesaver to prywatna PWA dla jednej osoby. Służy do codziennego śledzenia nawyków, zmiany niepożądanych zachowań i obserwacji długoterminowych postępów. Priorytety to prosta obsługa, rzetelne statystyki, dyskrecja i utrzymanie regularności przez wiele miesięcy.

## 2. Platformy i dostęp

- Interfejs projektujemy przede wszystkim na iPhone’a. Aplikacja działa też w przeglądarce na komputerze. Historia i ustawienia zwykłych nawyków synchronizują się między urządzeniami.
- Użytkownik loguje się kontem Google. Firebase Authentication i reguły Firestore chronią dane poza zakresem E2EE.
- E2EE obejmuje wyłącznie dwa nawyki strefy prywatnej i prywatny licznik, wraz z danymi, które ujawniają ich treść lub wyniki. Dostęp do tych danych wymaga osobnej blokady i sposobu odzyskiwania.
- Ekrany ogólnodostępne i powiadomienia ukrywają informacje prywatne.
- Preferowane są usługi darmowe, bez obowiązkowego abonamentu. PWA służy prywatnemu użytkowaniu. Ewentualne publiczne wydanie iOS w App Store ma być płatne jednorazowo, bez abonamentu. Cena i sposób dystrybucji technicznej pozostają otwarte. [Architektura](architektura.md) · [ADR-007](adr.md#adr-007)

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
| 8 | Zaśnięcie przed 1:00 | Zaśnięcie przed 01:00 |

- Wszystkie początkowe cele można później zmienić. Przy nauce zawodowej aplikacja zapisuje rzeczywisty czas, niezależnie od progu zaliczenia.
- Zaśnięcie i pobudka mają osobne karty i oceny w Dzienniku. Noc przypada na dzień pobudki. Każdą godzinę można wpisać, skorygować lub wyczyścić osobno. Przy domyślnych progach zaśnięcie poprzedniego dnia albo przed 01:00 w dniu pobudki zalicza cel. Pobudka zalicza cel przed 09:00. Same godziny 01:00 i 09:00 są poza celem.
- Czas TikToka można wprowadzać ręcznie. Użytkownik może później zmienić limit.

Dwa pozostałe nawyki startowe należą do strefy prywatnej. Ich konfiguracja pozostaje w specyfikacji lokalnej.

## 4. Prywatny licznik

**Status:** wymaganie do wdrożenia wraz ze strefą prywatną.

- Moduł rejestruje dzienną liczbę zdarzeń. Przyciski zwiększają i zmniejszają wartość, a historię można korygować.
- Jawne zero oznacza zarejestrowany wynik. Brak danych to osobny stan.
- Moduł pokazuje statystyki dzienne, tygodniowe i miesięczne, średnią dzienną oraz trend.
- Licznik służy obserwacji zachowań bez zawstydzania, limitów, automatycznej oceny i nagród za określony wynik. Pozostaje poza ogólnym wynikiem nawyków.

## 5. Ocena nawyków

| Stan | Znaczenie | Wpływ na realizację |
| --- | --- | --- |
| Tak | Warunek spełniony | 100% dla wpisu |
| Nie | Warunek niespełniony | 0% dla wpisu |
| Brak danych | Pole godziny lub innej wartości bez domyślnego pomiaru pozostaje puste | Pomijany w mianowniku realizacji |

W nawykach Tak/Nie domyślną odpowiedzią jest „Nie”. Można ją zmieniać na „Tak” i z powrotem. Czas trwania zaczyna od 0 min. Puste godziny, liczby wykonań i inne pomiary liczbowe pozostają bez oceny. Przy nawykach mierzonych wynik oblicza się z pomiaru i obowiązującego warunku. MVP pozwala poprawić pomiar, z którego wynika ocena. [ADR-008](adr.md#adr-008)

## 6. Statystyki i analiza postępów

- MVP pokazuje realizację, kompletność i zestawienia według nawyku dla wybranego dnia lub tygodnia oraz zbiorcze podsumowanie miesiąca.
- Realizacja = liczba zaliczonych pozycji ÷ liczba pozycji z wyliczoną oceną × 100%. Domyślne „Nie” i 0 min wpływają na realizację również przed zapisem.
- Kompletność = liczba faktycznie zapisanych wartości ÷ liczba zaplanowanych pozycji × 100%. Jawnie zapisane „Nie” i zero liczą się jako zapisane wartości.
- Przykład dla startowej listy w pustym dniu. Zero minut TikToka zalicza limit, pozostałe pięć pozycji z domyślną wartością daje „Nie”, a dwie puste godziny pomija się w ocenie. Wynik to 17% realizacji i 0% kompletności. Bez ocen realizacja jest niedostępna. Bez zaplanowanych pozycji nie ma także podstawy do obliczenia kompletności.
- Wszystkie codzienne nawyki mają równą wagę. Pełny wynik uwzględnia oba prywatne nawyki z zachowaniem osobnej blokady. Sposób prezentacji wyniku w widoku ogólnym pozostaje do ustalenia.
- Do wdrożenia pozostają porównania okresów, długoterminowe trendy i osobny ekran każdego nawyku z historią, wykresami i pomiarami. Widok szczegółowy nawyku jest wymagany. Filtrowanie według kategorii pozostaje poza zakresem v1.

## 7. Nawigacja i historia

- Pływający dolny pasek w stylu Liquid Glass otwiera „Dziennik”, „Historię” i „Nawyki”. Na ekranie dotykowym poziome przesunięcie przełącza sąsiednie zakładki.
- Historia pokazuje bieżący tydzień oraz statystyki wybranego dnia lub tygodnia. Kalendarz zawiera siedem pierścieni od poniedziałku do niedzieli i ósmy z wynikiem tygodnia. Wypełnienie oznacza realizację. Kalendarz można rozwinąć na pełny ekran, wybrać wcześniejszy tydzień albo przejść do miesiąca. Miesiąc pokazuje liczbę uzupełnionych pozycji pod każdą datą i otwiera Dziennik wybranego dnia. Bieżące okresy obejmują dni do dzisiaj włącznie.
- Szybkie przejście do dzisiaj i wczoraj oraz dostęp do wszystkich wcześniejszych dni.
- Można edytować zwykłe wpisy i godziny w całej historii. Statystyki przeliczają się po korekcie. Edycja historii licznika będzie częścią strefy prywatnej.
- Zaliczenie można zapisać wyłącznie dla bieżącej lub przeszłej daty aktywności.
- Data aktywności jest odrębna od daty wprowadzenia i ostatniej zmiany informacji.

## 8. Cele tygodniowe

Cele codzienne ocenia się osobno każdego dnia. Osobne cele tygodniowe są do wdrożenia. Będą rozliczane według liczby wykonań lub wartości, z własnymi wskaźnikami i statystykami. Początkowa lista będzie pusta. Obecne podsumowanie tygodnia agreguje nawyki codzienne.

## 9. Personalizacja

W MVP można dodawać własne nawyki typu Tak/Nie, minuty, liczba wykonań lub wartość liczbowa. Można zmieniać nazwy, dni tygodnia i progi, archiwizować i przywracać nawyki z zachowaniem wpisów oraz ustalać wspólną kolejność Dziennika. Zmiana definicji obowiązuje od wybranej daty, od dzisiaj lub później. Wcześniejsze dni zachowują poprzednie zasady. Trwałe usunięcie po potwierdzeniu pozostaje do wdrożenia.

## 10. Codzienne korzystanie i przypomnienia

- Wpis można zapisać od razu po wykonaniu lub uzupełnić zbiorczo później. Dziennik używa krótkich kart, aktywnego przycisku i pól pomiaru. Warunki zaliczenia są dostępne w zakładce „Nawyki”. Wybór Tak/Nie widać natychmiast, również podczas poprzedniego zapisu.
- Do wdrożenia jest konfigurowalne przypomnienie wieczorne o brakujących wpisach. Ma neutralną treść i pozostawia status wpisów bez zmian.
- Dodatkowe przypomnienia dla pojedynczych nawyków pozostają do ustalenia.

## 11. Filozofia produktu

Aplikacja pokazuje sukcesy, niepowodzenia i braki danych, zachowuje wcześniejszy postęp po gorszym dniu i wspiera porównania w czasie. Miarą postępu jest regularność i stopniowa poprawa. Użytkownik sam decyduje o rozkładzie dnia i czasie wolnym.

## 12. Zakres odłożony

Planowanie godzinowe, rozbudowane zarządzanie zadaniami i projektami, obowiązkowa integracja kalendarza, automatyczny odczyt wszystkich danych telefonu, pełne integracje Apple Health i Sleep Cycle, zaawansowana motywacja oraz domyślne cele tygodniowe pozostają poza wymaganym zakresem v1. Integracje snu można osobno ocenić technicznie, również pod kątem danych z Apple Watch.

## 13. Oczekiwany rezultat

Użytkownik może wygodnie i dyskretnie rejestrować dziesięć nawyków oraz prywatny licznik, poprawiać historię, analizować postępy i zmieniać własne cele na iPhonie i komputerze.
