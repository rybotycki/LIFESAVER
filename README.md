# Lifesaver

**Stan na 9 listopada 2026.** Prywatna PWA działa w środowisku testowym dla jednego właściciela. MVP obejmuje osiem zwykłych nawyków, edycję historii, zmianę kolejności, personalizację i osobne wskaźniki realizacji oraz kompletności. Doba jest liczona w strefie `Europe/Warsaw`. Czas trwania wpisuje się w pełnych minutach. [Specyfikacja](docs/specyfikacja-funkcjonalna.md) · [Architektura](docs/architektura.md) · [Model danych](docs/model-danych.md) · [ADR](docs/adr.md) · [Bezpieczeństwo](docs/bezpieczenstwo.md) · [Kwestie otwarte](docs/kwestie-otwarte.md)

„Zaśnięcie” i „Pobudka” to osobne pozycje na liście nawyków. Obie przypadają na dzień pobudki. Przy zaśnięciu wskaż poprzedni dzień albo dzień pobudki. Przy domyślnych progach zaśnięcie poprzedniego dnia lub przed 01:00 w dniu pobudki zalicza cel. Pobudka zalicza drugi cel przed 09:00. Godziny 01:00 i 09:00 są już poza celem. Każdą godzinę można zapisać, poprawić lub wyczyścić osobno.

Dolny pasek w stylu Liquid Glass otwiera „Dziennik”, „Historię” i „Nawyki”. Na ekranie dotykowym możesz przesunąć treść w lewo lub w prawo, aby przejść do sąsiedniego widoku. Karty Dziennika pokazują wynik za pomocą przycisków lub pól pomiaru. Progi są w zakładce „Nawyki”.

W „Dzienniku” wybierz „Zmień kolejność”, przesuń nawyki strzałkami i naciśnij „Zapisz kolejność”. Ustawienie obowiązuje we wszystkich dniach i synchronizuje się po zalogowaniu na to samo konto.

W zakładce „Nawyki” możesz dodać własny nawyk, zmienić nazwę, próg i dni tygodnia oraz przenieść nawyk do archiwum lub go przywrócić. Wybierz datę obowiązywania od dzisiaj lub późniejszą. Dziennik i podsumowania używają zasad obowiązujących dla każdej daty. Własne nawyki mogą mieć odpowiedź „Tak” lub „Nie”, pomiar w minutach, liczbę wykonań albo wartość liczbową.

Odpowiedzi Tak/Nie zaczynają od „Nie”, czas trwania od 0 min, a pola godzin, liczby wykonań i innych wartości liczbowych są puste. Domyślne odpowiedzi wpływają na realizację. Kompletność liczy wyłącznie zapisane wartości. Możesz zmienić „Tak” na „Nie” w trakcie zapisu. Wybór widać od razu, a aplikacja zapisuje go po zakończeniu poprzedniej operacji.

„Historia” pokazuje u góry bieżący tydzień. Siedem pierścieni odpowiada dniom od poniedziałku do niedzieli, a ósmy podsumowuje cały tydzień. Wypełnienie oznacza udział zaliczonych nawyków wśród ocenionych. Wyniki bieżącego tygodnia obejmują dni do dzisiaj włącznie. Dotknij pierścienia, aby zobaczyć statystyki dnia albo tygodnia. Z podsumowania dnia przycisk „Edytuj dzień” otwiera wpisy.

„Rozwiń kalendarz” otwiera pełnoekranowy widok wcześniejszych tygodni z przyciskami „Starsze” i „Nowsze”. Zakładka „Miesiąc” pokazuje podsumowanie, liczbę uzupełnionych pozycji pod każdą datą i przejście do wpisów wybranego dnia. Bieżący miesiąc obejmuje dni do dzisiaj włącznie.

## Środowisko testowe

MVP działa na Firebase Hosting z logowaniem Google i dostępem do dziennika ograniczonym do jednego właściciela. Firebase obsługuje zwykłe wpisy, sen oraz ustawienia nawyków. Tryb demonstracyjny zapisuje dane wyłącznie w przeglądarce. Bieżące repozytorium udostępnia dokumentację projektu.

### Wynik próby z 8 listopada 2026

Na komputerze potwierdzono logowanie, odczyt i zapis w dwóch kartach, odrzucenie nieaktualnej edycji i wylogowanie. Na iPhonie potwierdzono uruchomienie PWA z ekranu początkowego, logowanie i odczyt tego samego syntetycznego wpisu. Firestore odrzucił anonimowy odczyt i zapis. Odmowę dla obcego konta Google sprawdzono w symulatorze reguł.

### Weryfikacja z 9 listopada 2026

Sprawdzono zapis snu i kolejności, kalendarz miesiąca, ustawienia nawyków, Historię oraz przełączanie odpowiedzi w trakcie zapisu. Próby objęły mobilny widok Chrome i emulator Firestore. Aktualny kod przeszedł 13 lokalnych testów logiki.

## Dalszy zakres

Do wdrożenia pozostają strefa prywatna z E2EE, przypomnienie wieczorne, cele tygodniowe, osobne widoki nawyków, porównania i trendy oraz trwałe usuwanie nawyków. Tryb offline z synchronizacją oraz eksport i import kopii wymagają decyzji. PWA pozostaje prywatna. Ewentualne publiczne wydanie iOS w App Store ma być dostępne za jednorazową opłatą, bez abonamentu. [ADR](docs/adr.md#adr-007) · [Kwestie otwarte](docs/kwestie-otwarte.md)
