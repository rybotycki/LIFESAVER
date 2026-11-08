# Lifesaver — decyzje architektoniczne

**Zaakceptowana** oznacza decyzję użytkownika. **Proponowana** oznacza rozwiązanie do przeglądu. Wymagania produktowe opisuje [specyfikacja v1.0](specyfikacja-funkcjonalna.md).

| ADR | Decyzja | Status |
| --- | --- | --- |
| [001](#adr-001) | Zakres E2EE i dostęp do danych zwykłych | Zaakceptowana |
| [002](#adr-002) | Stos PWA i hosting | Proponowana |
| [003](#adr-003) | Klucze, blokada i odzyskiwanie | Proponowana |
| [004](#adr-004) | Model historii, statystyki i synchronizacja | Proponowana |
| [005](#adr-005) | Harmonogram przypomnień i Web Push | Proponowana |
| [006](#adr-006) | Zaszyfrowany eksport i przywracanie | Proponowana |

## ADR-001

**Zakres E2EE i dostęp do danych zwykłych · Zaakceptowana**

E2EE obejmuje wyłącznie dwa prywatne nawyki dotyczące materiałów seksualnych i prywatny licznik masturbacji. Dane poza tym zakresem chronią Firebase Authentication i reguły Firestore. Wariant szyfrowania całej historii został rozważony podczas ustalania zakresu. Wybrany podział pozwala prowadzić codzienny zapis bez dodatkowego odblokowania przy zwykłych wpisach. Sposób odblokowania, odzyskiwania i prezentacji pochodnych statystyk wymaga dopracowania. [Bezpieczeństwo](bezpieczenstwo.md)

## ADR-002

**Stos PWA i hosting · Proponowana**

React, TypeScript, Vite oraz Vite PWA, z Firebase Hosting jako hostingiem HTTPS. Wybrana już kontrola dostępu opiera się na Firebase Authentication i regułach Firestore. Zatwierdzenie pozostałego stosu zależy od próby instalacji, logowania Google w Safari i w zainstalowanej PWA oraz oszacowania kosztu dla jednego użytkownika. [Architektura](architektura.md)

## ADR-003

**Klucze, blokada i odzyskiwanie · Proponowana**

Szyfrowanie uwierzytelnione po stronie klienta przez Web Crypto API, losowy klucz danych i dwie niezależne drogi jego odblokowania. Proponowane są osobna fraza i losowy kod odzyskiwania przechowywany poza aplikacją. Do decyzji pozostają biblioteka, parametry, czas blokady, polityka urządzeń i rotacja. [Bezpieczeństwo](bezpieczenstwo.md)

## ADR-004

**Model historii, statystyki i synchronizacja · Proponowana**

Oddzielić definicję od wersji warunków i wpisu, a wyniki liczyć lokalnie. Zmiana celu miałaby obowiązywać od wybranej daty. Zapisy z dwóch urządzeń wymagałyby sprawdzania konfliktów. Szczegóły trwałej pracy offline i reguł przeliczania historii wymagają osobnej decyzji. [Model danych](model-danych.md)

## ADR-005

**Harmonogram przypomnień i Web Push · Proponowana**

Cloudflare Workers z Cron Trigger jako kandydat do harmonogramu, a FCM jako kandydat do neutralnej wysyłki. Decyzja zależy od dostawy na iPhone, rozwiązania problemu nieaktualnego stanu braków oraz określenia dostępu harmonogramu. Przy braku zgody lub wsparcia Web Push do oceny pozostaje przypomnienie wewnątrz aplikacji. [Architektura](architektura.md)

## ADR-006

**Zaszyfrowany eksport i przywracanie · Proponowana**

Ręczny eksport szyfrowanej kopii w wersjonowanym formacie i import z podglądem skutków. Kopia służy odtworzeniu historii po utracie lub przypadkowym usunięciu danych. Osobno trzeba ustalić ochronę kopii, dostęp po utracie urządzeń, zgodność wersji i sposób scalania przy imporcie. Szyfrowanie pliku kopii jest odrębną propozycją i pozostawia zakres E2EE synchronizacji zgodny z ADR-001. [Kwestie otwarte](kwestie-otwarte.md)
