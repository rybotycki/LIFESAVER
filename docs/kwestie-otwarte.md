# Lifesaver — kwestie otwarte

**Stan na 9 listopada 2026.** Poniżej są funkcje do wdrożenia, decyzje projektowe i brakujące próby. Bieżące zachowanie opisują [specyfikacja v1.0](specyfikacja-funkcjonalna.md) i [ADR](adr.md).

## Brakujące funkcje v1

| Temat | Zakres |
| --- | --- |
| Strefa prywatna | Dwa prywatne nawyki i prywatny licznik. E2EE, osobna blokada, odzyskiwanie, edycja historii i statystyki prywatne |
| Przypomnienie wieczorne | Ustawiana godzina i neutralna treść. Ustalić warunek wysyłki w przypadku nieaktualnych danych lub samych prywatnych braków oraz sposób ograniczania powtórzeń. FCM z Cloudflare Workers to kandydat do próby na iPhonie |
| Cele tygodniowe | Własne cele, próg, licznik postępu i osobne statystyki. Ustalić źródło postępu i regułę zmiany definicji w trakcie tygodnia |
| Statystyki | Osobna historia i wykres każdego nawyku, porównania okresów i trendy długoterminowe. Wybrać sposób pokazywania niepełnych okresów i braku odczytu |
| Trwałe usuwanie | Zaprojektować potwierdzenie, usuwanie wpisów i definicji na wielu urządzeniach oraz zachowanie wspólnego rekordu snu, statystyk i kopii |

## Decyzje techniczne i testy

| Temat | Pytanie do rozstrzygnięcia |
| --- | --- |
| Stos docelowy i koszty | React, TypeScript, Vite PWA i Firebase Hosting działają na stagingu. Ocenić zużycie odczytów, limity, koszty przypomnień i kopii oraz wymagania docelowego wdrożenia |
| Jedno konto i reguły | Ustalić uprawnienia administracyjne przyszłego środowiska. Odmowę dla obcego konta sprawdzono w symulatorze. Próbę na drugim rzeczywistym koncie odłożono. Ocenić, czy ograniczenie daty wpisu do dzisiaj lub wcześniej powinno być także egzekwowane przez bazę |
| Odblokowanie i odzyskiwanie | Jakie są format kluczy, wymagania frazy, parametry KDF, czas blokady, zachowanie w tle i zasady rotacji? Jak odtworzyć dostęp na drugim urządzeniu i co pokazać po utracie wszystkich metod odzyskania? |
| Dyskrecja widoku | Jak wyświetlić zwykłe i pełne wyniki bez ujawniania prywatnych danych? Jak dostarczyć początkowe definicje prywatne poza publicznym pakietem? Które dni wchodzą do średniej licznika? |
| Daty i oceny | Ustalić zachowanie przy podróży oraz przypisanie danych prywatnych do właściwej daty aktywności. Ustalić, czy umożliwić osobne potwierdzenie domyślnego „Nie” dla kompletności. Obecnie ponowne wybranie widocznej odpowiedzi nie zapisuje wartości |
| Offline i konflikty | Czy przechowywać niezapisane zmiany po zamknięciu PWA, kiedy włączyć trwałą pamięć i jak wykryć równoległe utworzenie prywatnego rekordu dla tej samej daty? |
| Kopia i import | Ustalić zakres eksportu, ochronę pliku i reguły scalania. Sprawdzić odtworzenie na drugim urządzeniu oraz niezgodność wersji |
| Próby na iPhonie | Instalacja, logowanie i odczyt podstawowego wpisu są potwierdzone. Wartości snu, nowe widoki Historii, personalizację, gesty i aktualizację PWA sprawdzano w kodzie lub próbach przeglądarkowych. Pełna próba najnowszych zmian na fizycznym iPhonie pozostaje do wykonania |

## Dalsze etapy

- Rozstrzygnąć termin, cenę i sposób realizacji ewentualnego wydania iOS. Capacitor, zapis najpierw na urządzeniu, zakres synchronizacji oraz Android pozostają otwarte. [ADR-007](adr.md#adr-007)
- Ocenić dodatkowe przypomnienia dla pojedynczych nawyków po wdrożeniu wieczornego przypomnienia.
- Osobno ocenić integracje Apple Health i Sleep Cycle, w tym dane z Apple Watch.
