# Lifesaver — architektura wysokiego poziomu

**Stan na 9 listopada 2026.** Poniżej opisano wdrożony MVP oraz propozycje na dalsze etapy. [Specyfikacja v1.0](specyfikacja-funkcjonalna.md) · [ADR](adr.md)

## Wdrożony zakres

React, TypeScript, Vite i Vite PWA obsługują mobilny Dziennik, Historię oraz Nawyki. MVP ma osiem zwykłych nawyków startowych, w tym dwie oceny oparte na wspólnym zapisie snu, oraz obsługę własnych definicji. Działa na Firebase Hosting z logowaniem Google przez Firebase Authentication i synchronizacją w Firestore. Na komputerze i w PWA uruchomionej z ekranu początkowego iPhone’a potwierdzono logowanie i odczyt tego samego wpisu. [Wyniki prób](../README.md#środowisko-testowe)

## Granice ustaleń

| Wymaganie | Środowisko testowe | Dalsze propozycje |
| --- | --- | --- |
| Prywatna PWA dla jednej osoby, projektowana przede wszystkim na telefon | React, TypeScript, Vite PWA z manifestem i service workerem oraz Firebase Hosting przez HTTPS | Ocena stosu dla docelowego wydania |
| Logowanie Google, kontrola dostępu i synchronizacja zwykłych wpisów | Firebase Authentication i Firestore z listą dostępu dla jednego właściciela. Rewizje wpisów i ustawienia obowiązujące od daty | Trwała praca offline i reguły scalania po synchronizacji |
| E2EE dwóch prywatnych nawyków oraz licznika, osobna blokada i odzyskiwanie | Do wdrożenia | Web Crypto API, format zaszyfrowanych rekordów, metoda ochrony kluczy i obsługa urządzeń |
| Konfigurowalne, dyskretne przypomnienie i preferencja darmowych usług | Do wdrożenia | Firebase Cloud Messaging (FCM) oraz Cloudflare Worker z Cron Trigger do wysyłki |

## Podział odpowiedzialności

| Element | Odpowiedzialność |
| --- | --- |
| PWA na każdym urządzeniu | Widoki, zapis wpisów, reguły zaliczania i obliczanie statystyk. Tryb demonstracyjny zapisuje dane tylko w przeglądarce. Przyszła strefa prywatna będzie szyfrować i odszyfrowywać dane na urządzeniu |
| Firebase Authentication | Logowanie Google i tożsamość konta właściciela |
| Cloud Firestore | Synchronizacja zwykłych wpisów, snu i ustawień nawyków. Reguły dostępu i walidacja formatu. Zapis zaszyfrowanych dokumentów prywatnych jest planowany |
| Firebase Hosting | Dostarczanie aplikacji po HTTPS. Adres pokazuje ekran logowania, a reguły Firestore ograniczają dostęp do dziennika do właściciela |
| Cloudflare Worker i FCM | Proponowany harmonogram i wysyłka neutralnych powiadomień z ograniczeniem powtórzeń |

Tryby demonstracyjny i Firebase używają wspólnego interfejsu zapisu i wspólnej logiki ocen. W planowanej strefie prywatnej klient ma szyfrować wpisy przed zapisem i odszyfrowywać je po osobnym odblokowaniu. Serwer otrzyma szyfrogramy z minimalną kopertą techniczną. [Model danych](model-danych.md) · [Bezpieczeństwo](bezpieczenstwo.md)

## Synchronizacja i obliczenia

- PWA subskrybuje wpisy i sen dla wybranych dat oraz osobno kolejność i zmiany definicji. Statystyki liczy lokalnie według wersji i harmonogramu obowiązujących w danym dniu. Doba jest liczona w `Europe/Warsaw`, tydzień zaczyna się w poniedziałek, a bieżące okresy kończą się dzisiaj. Realizacja sumuje oceny, również wynikające z wartości domyślnych. Kompletność uwzględnia zapisane wartości.
- Zwykły wpis, kolejność i zmiana definicji sprawdzają rewizję w transakcji. Sen porównuje poprzednią wartość edytowanego pola i zachowuje drugą godzinę. Próba konfliktu zwykłego wpisu w dwóch kartach potwierdziła odrzucenie starszej edycji. [Firestore — transakcje](https://firebase.google.com/docs/firestore/manage-data/transactions)
- Zapis transakcyjny wymaga połączenia. Tryb Firebase czeka na odczyt potwierdzony przez serwer. PWA przechowuje zasoby interfejsu przez service worker i informuje o gotowej aktualizacji. Trwała pamięć danych i kolejka offline pozostają do oceny. [Firestore — dostęp offline](https://firebase.google.com/docs/firestore/manage-data/enable-offline)

## Przypomnienia

- Proponowany Worker przelicza czas z harmonogramu UTC na strefę użytkownika, sprawdza stan przypomnienia i przekazuje FCM neutralną wiadomość, np. „Sprawdź dzisiejsze wpisy”. Wysyłka ma ograniczać ponowienia w tym samym dniu lokalnym. [Cloudflare — Cron Triggers](https://developers.cloudflare.com/workers/configuration/cron-triggers/) · [Firebase — odbieranie wiadomości w aplikacji webowej](https://firebase.google.com/docs/cloud-messaging/web/receive-messages)
- Ewentualny sygnał o brakach poza urządzeniem może pochodzić wyłącznie ze zwykłych wpisów, z uwzględnieniem pory dnia. Jeśli prywatne wpisy mają wpływać na przypomnienie, sposób obsługi wymaga osobnej decyzji bez ujawniania ich stanu przez treść, wysyłkę albo jej brak.
- Na iPhonie Web Push wymaga dodania aplikacji do ekranu początkowego i zgody wywołanej działaniem użytkownika. Dostawa powiadomienia przez FCM w zainstalowanej PWA wymaga osobnej próby. [WebKit — Web Push](https://webkit.org/blog/13878/web-push-for-web-apps-on-ios-and-ipados/)

## Koszt i dystrybucja

Środowisko testowe działa na planie Firebase Spark. Zużycie odczytów przy codziennym użyciu, koszty przypomnień i kopii oraz obsługa sekretów pozostają do weryfikacji. [Firebase — plany](https://firebase.google.com/docs/projects/billing/firebase-pricing-plans) · [Firestore — limity](https://firebase.google.com/docs/firestore/quotas) · [Cloudflare — plany](https://developers.cloudflare.com/workers/platform/pricing/)

PWA pozostaje prywatna. Kierunek ewentualnego wydania dla innych użytkowników to płatna aplikacja iOS w App Store, z jednorazowym zakupem bez abonamentu i ceną nastawioną na pokrycie kosztów. Współdzielenie kodu, Capacitor, zapis najpierw na urządzeniu z opcjonalną synchronizacją oraz wydanie na Androida pozostają propozycjami. Model pojedynczego właściciela dotyczy prywatnej PWA. Zakres obsługi wielu użytkowników wymagałby osobnego projektu. [ADR-007](adr.md#adr-007) · [Wytyczne App Store](https://developer.apple.com/app-store/review/guidelines/)
