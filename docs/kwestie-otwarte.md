# Lifesaver — kwestie otwarte

Każdy punkt wymaga decyzji albo próby technicznej. Zatwierdzone wymagania opisuje [specyfikacja v1.0](specyfikacja-funkcjonalna.md), a status propozycji jest w [ADR](adr.md).

## Przed zatwierdzeniem architektury

| Temat | Pytanie do rozstrzygnięcia |
| --- | --- |
| Stos i hosting | Czy przyjąć React, TypeScript, Vite PWA i Firebase Hosting? Sprawdzić instalację PWA oraz logowanie Google na iPhonie i komputerze, w tym w trybie zainstalowanej PWA |
| Jedno konto i środowiska | Jak dopuścić wyłącznie konto właściciela, zarządzać uprawnieniami administracyjnymi oraz oddzielić dane testowe od rzeczywistych? |
| Odblokowanie i odzyskiwanie | Jakie są format kluczy, wymagania frazy, parametry KDF, czas blokady, zachowanie w tle i zasady rotacji? Jak odtworzyć dostęp na drugim urządzeniu i co pokazać po utracie wszystkich metod odzyskania? |
| Dyskrecja widoku | Czy na ekranie ogólnym pokazać podpisane statystyki zwykłych nawyków, czy ukryć wspólny wynik? Jak bezpiecznie dostarczyć początkowe definicje prywatne bez umieszczania ich w publicznym pakiecie aplikacji? |
| Powiadomienia | Czy FCM z Cloudflare Workers daje akceptowalną dostawę na iPhonie? Ustalić neutralny mechanizm przy nieaktualnym stanie, braku zgody albo samych prywatnych brakach, granice dostępów i zapobieganie duplikatom |
| Koszty | Czy wybrany zakres odczytów, hosting i przypomnienia mieszczą się w darmowych planach? Sprawdzić zachowanie po przekroczeniu limitu i konsekwencje wyboru kopii zapasowych |

## Przed utrwaleniem modelu i reguł produktu

| Temat | Pytanie do rozstrzygnięcia |
| --- | --- |
| Granice doby, nocy i tygodnia | Ustalić strefę i zachowanie przy podróży oraz zmianie czasu, początek tygodnia i interpretację zaśnięcia przed 01:00 dla nocy przypisanej do dnia pobudki. Doprecyzować, do którego dnia należy przedział 00:00–zaśnięcie, także przy zaśnięciu przed północą |
| Oceny i historia celów | Która wartość rozstrzyga przy ręcznej zmianie statusu sprzecznej z pomiarem? Czy zmiana progu działa od wskazanego dnia? Jak traktować dni przed dodaniem nawyku oraz po archiwizacji i przywróceniu? |
| Statystyki | Potwierdzić prezentację kompletności przy zerze ocen i dodatnim mianowniku. Określić, które dni wchodzą do średniej prywatnego licznika i jak prezentować niepełny okres oraz niekompletną synchronizację |
| Cele tygodniowe | Czy postęp pochodzi z ręcznego wpisu, z rejestru zdarzeń czy z agregacji dziennych zapisów? Jak zmienić definicję w trakcie tygodnia? |
| Offline i konflikty | Jak długo przetrwa niezapisana zmiana po zamknięciu PWA? Czy i kiedy włączyć trwałą pamięć na zaufanym urządzeniu? Jak wykrywać równoległe edycje oraz podwójne utworzenie prywatnego rekordu dla tego samego dnia? |
| Trwałe usuwanie | Jak usuwać historię na wielu urządzeniach i blokować przywrócenie usuniętych rekordów przez opóźnione zapisy? Jak potraktować wspólny zapis snu, statystyki i istniejące kopie? |
| Kopia i import | Co obejmuje eksport, jak chronić plik i obsłużyć scalanie albo zastąpienie danych? Sprawdzić odtworzenie na drugim urządzeniu i niezgodność wersji |

## Szczegóły interfejsu i dalsze możliwości

- Wybrać format kalendarza, wykresów i porównań, w tym prezentację braków danych.
- Ocenić dodatkowe przypomnienia per nawyk po sprawdzeniu wieczornego przypomnienia.
- Osobno zweryfikować możliwości integracji Apple Health i Sleep Cycle, w tym dostępność danych z Apple Watch.
