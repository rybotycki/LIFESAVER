# Lifesaver — model danych

**Stan na 9 listopada 2026.** Pierwsza sekcja opisuje zapis w działającym MVP. Dalsze encje i kolekcje to propozycje. [Specyfikacja v1.0](specyfikacja-funkcjonalna.md) · [ADR-001](adr.md#adr-001)

## Zapis używany w środowisku testowym

| Dokument | Pola i znaczenie |
| --- | --- |
| `users/{uid}/dailyEntries/{activityDate}_{habitId}` | `activityDate`, `habitId`, `value`. Wartość to odpowiedź logiczna, liczba lub `null`. |
| `users/{uid}/sleepRecords/{activityDate}` | `bedtimeMinutes` i `wakeMinutes`. Kluczem jest dzień pobudki. Każde pole może być `null`. |
| `users/{uid}/settings/habits` | Lista `ids` z kolejnością Dziennika dla wszystkich dni. Klient uzupełnia brakujące aktywne pozycje w zapisanej kolejności. |
| `users/{uid}/habitChanges/{habitId}_{since}` | `habitId`, `since`, `name`, `enabled`, `weekdays`, `kind`, `limit`, `unit`. Zmiana definiuje nawyk od daty `since`. Własny nawyk ma stabilny identyfikator `custom-<UUID>`. |

Wszystkie te dokumenty mają `revision`, `createdAt` i `updatedAt`. Pierwszy zapis otrzymuje rewizję `1`. Każda korekta podnosi ją o `1`, zachowuje `createdAt` i aktualizuje `updatedAt` na serwerze. Zwykły wpis, kolejność i zmiana definicji porównują rewizję. Sen porównuje poprzednią wartość edytowanego pola i zachowuje aktualną wartość drugiego pola. [Synchronizacja](architektura.md#synchronizacja-i-obliczenia)

Daty aktywności używają `YYYY-MM-DD` w strefie `Europe/Warsaw`. Godziny snu to minuty zegarowe, więc model przechowuje je bez osobnych znaczników czasu zdarzenia. Dla zaśnięcia poprzedniego dnia odejmuje się 1440 od liczby minut od północy. Gdy zapisano obie godziny, zaśnięcie musi poprzedzać pobudkę. Zbiorczy odczyt snu jest ograniczony do 31 dokumentów na zapytanie.

Brak dokumentu i `null` oznaczają brak zapisanej wartości. Klient stosuje domyślne `false` dla Tak/Nie i `0` dla czasu trwania. Wprost zapisane `false` i `0` wchodzą do kompletności. Brak godziny, liczby wykonań lub innego pomiaru liczbowego pozostaje nieoceniony. Minuty i liczby wykonań są całkowite w interfejsie. Ocena powstaje lokalnie z wartości i warunku obowiązującego dla daty aktywności.

Dla każdego dnia aplikacja wybiera najnowszą zmianę obowiązującą w tej dacie. Dni tygodnia mają wartości od 0 (niedziela) do 6 (sobota). Archiwizacja zapisuje definicję z `enabled: false`, a przywrócenie włącza ją od wybranej daty. Obie operacje zachowują pomiary. Statystyki sumują wyniki i zaplanowane pozycje ze wszystkich uwzględnionych dni.

## Docelowe encje domenowe

| Encja | Kluczowe pola i przeznaczenie |
| --- | --- |
| `UserSettings` | Strefa czasowa IANA, ustawienia aplikacji i godzina przypomnienia |
| `Habit` | Stały identyfikator, nazwa, typ pomiaru i okresy aktywności. Definicje prywatne w E2EE |
| `HabitVersion` | Identyfikator nawyku, data wejścia w życie, częstotliwość, jednostka i warunek zaliczenia, np. próg w minutach. Wersje nawyków prywatnych w E2EE |
| `DailyEntry` | Data aktywności, nawyk, wersja reguły, opcjonalny pomiar i status. Wpisy prywatne w E2EE |
| `SleepRecord` | Jeden rekord na dzień pobudki zasila obie oceny. Ewentualne przejście z minut zegarowych na pełne znaczniki czasu wymaga decyzji |
| `WeeklyGoal`, `WeeklyProgress` | Definicja celu, jednostka i próg. Postęp przypisany do tygodnia. Osobne statystyki od dziennych |
| `PrivateCounterEntry` | Data aktywności, nieujemna liczba całkowita oraz znaczniki edycji. Całość w E2EE |
| `KeySettings` | Wersja formatu i klucza, parametry odblokowania oraz zaszyfrowane kopie klucza danych |
| `NotificationSettings` | Zgoda i identyfikator subskrypcji urządzenia oraz dane niezbędne do zaplanowania i ograniczenia powtórzeń |

## Proponowany układ w Firestore

| Ścieżka lub grupa dokumentów | Zawartość |
| --- | --- |
| `users/{uid}` | Zwykłe ustawienia właściciela. UID z Firebase Authentication |
| Kolekcje `weeklyGoals` i `weeklyProgress` pod właścicielem | Proponowane definicje i postęp celu tygodniowego. Osobny zapis dla pary cel–tydzień, jeśli postęp będzie wpisywany ręcznie |
| `users/{uid}/privateRecords/{opaqueId}` | Wspólna kolekcja zaszyfrowanych definicji, wersji, wpisów i licznika. Neutralny, losowy identyfikator. Typ, data aktywności, nazwa i status wewnątrz szyfrogramu |
| `users/{uid}/keySettings/{keyId}` | Materiał potrzebny do odblokowania lub odzyskania. Klucz w postaci zaszyfrowanej |
| Stan harmonogramu i subskrypcji push | Proponowany odrębny magazyn lub wąski interfejs, z dostępem ograniczonym do obsługi przypomnień. Wybór lokalizacji pozostaje otwarty |

Wstępna koperta prywatnego rekordu zawiera wersję formatu, odniesienie do wersji klucza, nonce i szyfrogram z tagiem uwierzytelniającym. Ewentualna jawna rewizja na potrzeby synchronizacji wymaga oceny prywatności. Klient rozpoznaje rodzaj rekordu i jego datę po odszyfrowaniu. Trzeba ustalić sposób zachowania jednego wpisu na nawyk i dzień przy równoległym utworzeniu rekordów z losowymi identyfikatorami. [Bezpieczeństwo](bezpieczenstwo.md) · [Kwestie otwarte](kwestie-otwarte.md)

## Daty, wersje i reguły

- Odseparować stan braku odczytu lub blokady prywatnej od braku zapisanej wartości. Zero na prywatnym liczniku jest zapisaną wartością.
- Trwałe usunięcie po potwierdzeniu ma objąć definicję, jej wersje i przypisane wpisy. Do ustalenia pozostaje wpływ na wspólny rekord snu, statystyki historyczne, kopie oraz oczekujące zapisy na drugim urządzeniu. Firestore wymaga jawnego usunięcia dokumentów powiązanych, w tym dokumentów podkolekcji. [Firestore — usuwanie danych](https://firebase.google.com/docs/firestore/manage-data/delete-data)
- Plan zakłada obliczanie statystyk prywatnych i wspólnych podsumowań na urządzeniu po odblokowaniu. Licznik i cele tygodniowe będą miały osobne statystyki.
