# Lifesaver — decyzje architektoniczne

**Zaakceptowana** oznacza decyzję użytkownika. **Wdrożona w MVP** oznacza stan środowiska testowego. **Proponowana** oznacza rozwiązanie do przeglądu. [Specyfikacja v1.0](specyfikacja-funkcjonalna.md)

| ADR | Decyzja | Status |
| --- | --- | --- |
| [001](#adr-001) | Zakres E2EE i dostęp do danych zwykłych | Zaakceptowana |
| [002](#adr-002) | Stos PWA i hosting | Wdrożona w MVP. Stos docelowy otwarty |
| [003](#adr-003) | Klucze, blokada i odzyskiwanie | Proponowana |
| [004](#adr-004) | Wersje nawyków, historia i synchronizacja | Wdrożona w MVP. Offline otwarte |
| [005](#adr-005) | Harmonogram przypomnień i Web Push | Proponowana |
| [006](#adr-006) | Zaszyfrowany eksport i przywracanie | Proponowana |
| [007](#adr-007) | Prywatna PWA i model dystrybucji iOS | Zaakceptowana |
| [008](#adr-008) | Wartości domyślne, realizacja i kompletność | Zaakceptowana i wdrożona w MVP |

## ADR-001

**Zakres E2EE i dostęp do danych zwykłych · Zaakceptowana**

E2EE obejmuje wyłącznie dwa nawyki strefy prywatnej i prywatny licznik. Dane poza tym zakresem chronią Firebase Authentication i reguły Firestore. Wariant szyfrowania całej historii został rozważony podczas ustalania zakresu. Wybrany podział pozwala prowadzić codzienny zapis bez dodatkowego odblokowania przy zwykłych wpisach. Sposób odblokowania, odzyskiwania i prezentacji pochodnych statystyk wymaga dopracowania. [Bezpieczeństwo](bezpieczenstwo.md)

## ADR-002

**Stos PWA i hosting · Wdrożona w MVP**

React, TypeScript, Vite i Vite PWA z Firebase Hosting oraz Firebase Authentication działają w środowisku testowym. Próba z 8 listopada 2026 potwierdziła logowanie i odczyt wpisu na komputerze oraz w zainstalowanej PWA na iPhonie. Decyzja o stosie docelowym i ocena kosztów przy codziennym użyciu pozostają otwarte. [Wynik próby](../README.md#wynik-próby-z-8-listopada-2026)

## ADR-003

**Klucze, blokada i odzyskiwanie · Proponowana**

Szyfrowanie uwierzytelnione po stronie klienta przez Web Crypto API, losowy klucz danych i dwie niezależne drogi jego odblokowania. Proponowane są osobna fraza i losowy kod odzyskiwania przechowywany poza aplikacją. Do decyzji pozostają biblioteka, parametry, czas blokady, polityka urządzeń i rotacja. [Bezpieczeństwo](bezpieczenstwo.md)

## ADR-004

**Wersje nawyków, historia i synchronizacja · Wdrożona w MVP**

MVP przechowuje pomiary i zmiany definicji oddzielnie. Ustawienia obowiązują od wybranej daty, a historia korzysta z definicji właściwej dla danego dnia. Statystyki są liczone lokalnie. Transakcje chronią przed zapisem na podstawie nieaktualnej rewizji, a zapis snu porównuje poprzednią wartość edytowanego pola. Trwała praca offline i konflikty zaszyfrowanych rekordów pozostają otwarte. [Model danych](model-danych.md)

## ADR-005

**Harmonogram przypomnień i Web Push · Proponowana**

Cloudflare Workers z Cron Trigger jako kandydat do harmonogramu, a FCM jako kandydat do neutralnej wysyłki. Decyzja zależy od dostawy na iPhone, rozwiązania problemu nieaktualnego stanu braków oraz określenia dostępu harmonogramu. Przy braku zgody lub wsparcia Web Push do oceny pozostaje przypomnienie wewnątrz aplikacji. [Architektura](architektura.md)

## ADR-006

**Zaszyfrowany eksport i przywracanie · Proponowana**

Ręczny eksport szyfrowanej kopii w wersjonowanym formacie i import z podglądem skutków. Kopia służy odtworzeniu historii po utracie lub przypadkowym usunięciu danych. Osobno trzeba ustalić ochronę kopii, dostęp po utracie urządzeń, zgodność wersji i sposób scalania przy imporcie. Szyfrowanie pliku kopii jest odrębną propozycją i pozostawia zakres E2EE synchronizacji zgodny z ADR-001. [Kwestie otwarte](kwestie-otwarte.md)

## ADR-007

**Prywatna PWA i model dystrybucji iOS · Zaakceptowana**

PWA pozostaje prywatna i służy jednemu właścicielowi. Jeżeli Lifesaver trafi do publicznej dystrybucji na iOS, aplikacja w App Store ma mieć jednorazową cenę zakupu, bez abonamentu, z celem pokrycia kosztów. Cena, termin wydania, użycie Capacitor, zapis najpierw na urządzeniu z opcjonalną synchronizacją i ewentualna wersja Android pozostają otwarte. [Architektura](architektura.md#koszt-i-dystrybucja)

## ADR-008

**Wartości domyślne, realizacja i kompletność · Zaakceptowana i wdrożona w MVP**

Nawyki Tak/Nie zaczynają od „Nie”, pola czasu trwania od 0 min, a pola godzin pozostają puste. Domyślne wartości wpływają na realizację. Kompletność liczy zapisane wartości. Brak pomiaru innego niż czas trwania pozostaje nieoceniony. Zmiana uzupełnia pierwotne założenie o „Braku danych” jako domyślnym stanie każdego nawyku. [Ocena nawyków](specyfikacja-funkcjonalna.md#5-ocena-nawyków)
