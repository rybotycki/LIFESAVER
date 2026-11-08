# Lifesaver — wstępny model danych

**Status:** propozycja. Podział na dane zwykłe i prywatne wynika z [ADR-001](adr.md#adr-001). Poniższe encje i reguły zapisu rozwijają [specyfikację v1.0](specyfikacja-funkcjonalna.md).

## Encje domenowe

| Encja | Kluczowe pola i przeznaczenie |
| --- | --- |
| `UserSettings` | Strefa czasowa IANA, ustawienia aplikacji i godzina przypomnienia |
| `Habit` | Stały identyfikator, nazwa, typ pomiaru i okresy aktywności. Definicje prywatne w E2EE |
| `HabitVersion` | Identyfikator nawyku, data wejścia w życie, częstotliwość, jednostka i warunek zaliczenia, np. próg w minutach. Wersje nawyków prywatnych w E2EE |
| `DailyEntry` | Data aktywności, nawyk, wersja reguły, opcjonalny pomiar i status. Wpisy prywatne w E2EE |
| `SleepRecord` | Data pobudki jako data aktywności, znaczniki zaśnięcia i pobudki, strefa zapisu. Jeden rekord może zasilać obie oceny snu |
| `WeeklyGoal`, `WeeklyProgress` | Definicja celu, jednostka i próg. Postęp przypisany do tygodnia. Osobne statystyki od dziennych |
| `PrivateCounterEntry` | Data aktywności, nieujemna liczba całkowita oraz znaczniki edycji. Całość w E2EE |
| `KeySettings` | Wersja formatu i klucza, parametry odblokowania oraz zaszyfrowane kopie klucza danych |
| `NotificationSettings` | Zgoda i identyfikator subskrypcji urządzenia oraz dane niezbędne do zaplanowania i ograniczenia powtórzeń |

## Proponowany układ w Firestore

| Ścieżka lub grupa dokumentów | Zawartość |
| --- | --- |
| `users/{uid}` | Zwykłe ustawienia właściciela. UID z Firebase Authentication |
| `users/{uid}/habits/{habitId}` i kolekcja `habitVersions` pod właścicielem | Zwykłe definicje i wersje. Każda wersja wskazuje nawyk |
| Kolekcje `dailyEntries`, `sleepRecords`, `weeklyGoals` i `weeklyProgress` pod właścicielem | Zwykłe wpisy. Osobny dokument dla pary nawyk–dzień, jeden rekord snu na dzień pobudki i osobny zapis postępu dla pary cel–tydzień, o ile postęp jest wpisywany ręcznie |
| `users/{uid}/privateRecords/{opaqueId}` | Wspólna kolekcja zaszyfrowanych definicji, wersji, wpisów i licznika. Neutralny, losowy identyfikator. Typ, data aktywności, nazwa i status wewnątrz szyfrogramu |
| `users/{uid}/keySettings/{keyId}` | Materiał potrzebny do odblokowania lub odzyskania. Klucz w postaci zaszyfrowanej |
| Stan harmonogramu i subskrypcji push | Proponowany odrębny magazyn lub wąski interfejs, z dostępem ograniczonym do obsługi przypomnień. Wybór lokalizacji pozostaje otwarty |

Wstępna koperta prywatnego rekordu zawiera wersję formatu, odniesienie do wersji klucza, nonce i szyfrogram z tagiem uwierzytelniającym. Ewentualna jawna rewizja na potrzeby synchronizacji wymaga oceny prywatności. Klient rozpoznaje rodzaj rekordu i jego datę po odszyfrowaniu. Trzeba ustalić sposób zachowania jednego wpisu na nawyk i dzień przy równoległym utworzeniu rekordów z losowymi identyfikatorami. [Bezpieczeństwo](bezpieczenstwo.md) · [Kwestie otwarte](kwestie-otwarte.md)

## Daty, wersje i reguły

- `activityDate` to lokalny dzień w formacie `YYYY-MM-DD`. Dla zdarzeń ze znaną godziną używamy znacznika czasu i strefy IANA, a czas utworzenia i aktualizacji ma osobne pola. Granice dnia, zmiana czasu i podróże wymagają wspólnej polityki.
- Przy nawyku mierzonym status wynika z pomiaru i wersji warunku. Propozycja zachowuje historyczny warunek po zmianie celu. Zmiana limitu na przyszłość i korekta wcześniejszego pomiaru to odrębne operacje. Zasady ręcznej korekty statusu wymagają decyzji.
- `yes` i `no` to oceny. Brak dokumentu lub zapis bez oceny reprezentuje „Brak danych”. Brak odczytu pełnej historii lub blokada prywatna to odrębne stany aplikacji. Zero minut i zero na liczniku są wartościami.
- Archiwizacja zamyka okres aktywności, a przywrócenie otwiera kolejny. Mianownik kompletności wynika z harmonogramu aktywnych nawyków w danym okresie. Statystyki liczy się ze zsumowanych wpisów, bez uśredniania procentów dni o różnych mianownikach.
- Trwałe usunięcie po potwierdzeniu obejmuje definicję, jej wersje i przypisane wpisy. Projekt musi określić wpływ na wspólny rekord snu, statystyki historyczne, kopie oraz oczekujące zapisy na drugim urządzeniu. Firestore wymaga jawnego usunięcia dokumentów powiązanych, w tym dokumentów podkolekcji. [Firestore — usuwanie danych](https://firebase.google.com/docs/firestore/manage-data/delete-data)
- Statystyki prywatne i wspólne podsumowania uwzględniające prywatne nawyki powstają na urządzeniu po odblokowaniu. Licznik i cele tygodniowe mają własne statystyki.
