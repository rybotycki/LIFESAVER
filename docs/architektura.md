# Lifesaver — architektura wysokiego poziomu

**Status:** propozycja do przeglądu. Źródłem wymagań jest [specyfikacja v1.0](specyfikacja-funkcjonalna.md), a statusy decyzji są w [ADR](adr.md).

## Granice ustaleń

| Ustalone | Proponowane |
| --- | --- |
| PWA dla jednej osoby na iPhonie i komputerze, logowanie Google i synchronizacja | React, TypeScript, Vite oraz Vite PWA z manifestem i service workerem |
| Firebase Authentication i reguły Firestore do kontroli dostępu do danych zwykłych | Firebase Hosting jako statyczny hosting HTTPS, układ kolekcji i sposób synchronizacji |
| E2EE dwóch prywatnych nawyków oraz licznika, osobna blokada i odzyskiwanie | Web Crypto API, format zaszyfrowanych rekordów, metoda ochrony kluczy i obsługa urządzeń |
| Konfigurowalne, dyskretne przypomnienie i preferencja darmowych usług | Firebase Cloud Messaging (FCM) oraz Cloudflare Worker z Cron Trigger do wysyłki |

## Podział odpowiedzialności

| Element | Odpowiedzialność |
| --- | --- |
| PWA na każdym urządzeniu | Widoki, zapis wpisów, reguły zaliczania i obliczanie statystyk. W prywatnej części także szyfrowanie i odszyfrowanie na urządzeniu |
| Firebase Authentication | Logowanie Google i tożsamość konta właściciela |
| Cloud Firestore | Synchronizacja zwykłych wpisów i zaszyfrowanych dokumentów prywatnych. Reguły dostępu i walidacja formatu |
| Firebase Hosting | Dostarczanie aplikacji po HTTPS |
| Cloudflare Worker i FCM | Opcjonalna część techniczna przypomnień. Harmonogram, wysyłka neutralnej wiadomości oraz ograniczenie powtórzeń |

Po zalogowaniu PWA pobiera dane właściciela. Zwykłe wpisy może odczytać od razu. Prywatne wpisy szyfruje przed zapisem i odszyfrowuje lokalnie po osobnym odblokowaniu. Do Firestore trafiają szyfrogramy z minimalną kopertą techniczną. Widok ogólny pozostaje odseparowany od odblokowanego widoku prywatnego. [Model danych](model-danych.md) · [Bezpieczeństwo](bezpieczenstwo.md)

## Synchronizacja i obliczenia

- Propozycja zakłada wersje warunków zaliczenia oraz obliczanie statystyk z zapisów źródłowych na urządzeniu. Korekta historii zmienia wynik po przeliczeniu.
- Firestore obsługuje kolejkę zapisu i synchronizację po powrocie połączenia. Trwała pamięć podręczna w aplikacji webowej wymaga jawnej konfiguracji. Proponowany start to pamięć sesji i widoczny stan zapisu. Trwały zapis na zaufanym urządzeniu pozostaje do oceny. [Firestore — dostęp offline](https://firebase.google.com/docs/firestore/manage-data/enable-offline)
- Domyślne rozstrzyganie zapisu tego samego dokumentu przez Firestore może nadpisać konkurencyjną zmianę. Propozycja przewiduje sprawdzenie wersji rekordu podczas synchronizacji oraz czytelne pokazanie konfliktu. Zakres kolejki trwałej i sposób rozstrzygania konfliktu pozostają otwarte. [Firestore — dostęp offline](https://firebase.google.com/docs/firestore/manage-data/enable-offline) · [Firestore — transakcje](https://firebase.google.com/docs/firestore/manage-data/transactions)

## Przypomnienia

- Proponowany Worker przelicza czas z harmonogramu UTC na strefę użytkownika, sprawdza stan przypomnienia i przekazuje FCM neutralną wiadomość, np. „Sprawdź dzisiejsze wpisy”. Wysyłka ma ograniczać ponowienia w tym samym dniu lokalnym. [Cloudflare — Cron Triggers](https://developers.cloudflare.com/workers/configuration/cron-triggers/) · [Firebase — odbieranie wiadomości w aplikacji webowej](https://firebase.google.com/docs/cloud-messaging/web/receive-messages)
- Ewentualny sygnał o brakach poza urządzeniem może pochodzić wyłącznie ze zwykłych wpisów, z uwzględnieniem pory dnia. Jeśli prywatne wpisy mają wpływać na przypomnienie, sposób obsługi wymaga osobnej decyzji bez ujawniania ich stanu przez treść, wysyłkę albo jej brak.
- Na iPhonie Web Push wymaga dodania aplikacji do ekranu początkowego i zgody wywołanej działaniem użytkownika. Wybór FCM i konfigurację logowania Google trzeba zweryfikować na docelowych urządzeniach, w tym w trybie zainstalowanej PWA. [WebKit — Web Push](https://webkit.org/blog/13878/web-push-for-web-apps-on-ios-and-ipados/) · [Firebase — logowanie w przeglądarkach ograniczających pamięć stron trzecich](https://firebase.google.com/docs/auth/web/redirect-best-practices)

## Koszt i kolejność weryfikacji

Plan Firebase Spark i darmowy plan Cloudflare Workers są punktem wyjścia do oszacowania kosztu. Cel to brak obowiązkowego abonamentu. Limity, zużycie odczytów, sposób obsługi sekretów i ewentualne płatne elementy należy sprawdzić przed wyborem dostawców. [Firebase — plany](https://firebase.google.com/docs/projects/billing/firebase-pricing-plans) · [Firestore — limity](https://firebase.google.com/docs/firestore/quotas) · [Cloudflare — plany i limity rozliczeń](https://developers.cloudflare.com/workers/platform/pricing/)

Proponowana kolejność po zatwierdzeniu decyzji:

1. Sprawdzić logowanie, instalację PWA, synchronizację i testową dostawę push na iPhonie i komputerze.
2. Zweryfikować model dostępu, E2EE, odblokowanie i odzyskiwanie na drugim urządzeniu.
3. Zaprojektować tracker i historię, potem statystyki, personalizację i cele tygodniowe.
4. Domknąć przypomnienia, zasady kopii i testy korekt historii na kilku urządzeniach.
