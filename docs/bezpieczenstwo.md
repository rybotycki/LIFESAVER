# Lifesaver — bezpieczeństwo i prywatność

**Stan na 9 listopada 2026.** Dostęp do zwykłych danych działa w środowisku testowym. Zakres E2EE jest zaakceptowany, a szyfrowanie, blokada i odzyskiwanie pozostają do wdrożenia. [ADR-001](adr.md#adr-001) · [ADR-003](adr.md#adr-003)

## Klasy danych i granice zaufania

| Klasa | Zakres | Ochrona |
| --- | --- | --- |
| Zwykłe dane konta | Zwykłe nawyki, sen, pomiary, cele tygodniowe i ogólne ustawienia | Logowanie Google przez Firebase Authentication i reguły Firestore. Usługa przetwarza treść po stronie serwera |
| Planowane E2EE | Dwa prywatne nawyki oraz prywatny licznik, wraz z ich nazwami, regułami, datami, stanami i pomiarami | Szyfrowanie na urządzeniu przed zapisem. Odczyt treści po osobnym odblokowaniu |
| Metadane i obsługa usług | UID właściciela, liczba i rozmiar szyfrogramów, techniczne czasy zapisu, subskrypcje push | Minimalizacja pól, uprawnień i czasu przechowywania |

Pełny wynik i kompletność mogą ujawnić stan prywatnych wpisów przez porównanie z widokiem zwykłym. Propozycja przewiduje obliczanie pełnego wyniku tylko po odblokowaniu, w strefie prywatnej. Do ustalenia pozostaje, czy widok ogólny pokazuje wyraźnie opisaną statystykę zwykłych nawyków, czy pomija wspólny wynik.

## Dostęp do Firestore

- Reguły w środowisku testowym dopuszczają odczyt, tworzenie i aktualizację wpisów, snu oraz ustawień nawyków przez zalogowane konto Google, którego UID zgadza się ze ścieżką i polem `uid` w dokumencie `access/owner`. Administrator konfiguruje ten dokument w konsoli. Klient nie ma do niego dostępu. Adres hostingu pokazuje ekran logowania, a dostęp do danych dziennika ograniczają reguły bazy.
- Reguły sprawdzają zestaw pól, identyfikatory, dopuszczalne wartości, rewizję i czas zapisu. Klient ogranicza wybór daty wpisu do dzisiaj lub wcześniej. Reguły bazy sprawdzają format daty. Przetestowano operacje właściciela i odmowę anonimowego odczytu i zapisu. Odmowę dla obcego konta Google sprawdzono w symulatorze. Próbę na drugim rzeczywistym koncie odłożono zgodnie z decyzją użytkownika. [Wyniki prób](../README.md#wynik-próby-z-8-listopada-2026)
- W docelowej kolekcji prywatnej reguły mają dopuszczać tylko kopertę techniczną, a treść ma walidować klient po odszyfrowaniu. Pozostają testy prób zapisu jawnych pól, ustalenie uprawnień IAM oraz ograniczenie dostępu harmonogramu do stanu wysyłki i subskrypcji. [Firebase — reguły i uwierzytelnianie](https://firebase.google.com/docs/rules/rules-and-auth) · [Firestore — kontrola dostępu](https://firebase.google.com/docs/firestore/security/get-started)

## Proponowany cykl życia klucza

1. PWA generuje losowy klucz danych. Web Crypto API szyfruje rekordy przy użyciu AES-256-GCM, z nonce unikalnym dla danego klucza przy każdym szyfrowaniu. Dane uwierzytelniane mogą wiązać szyfrogram z właścicielem, identyfikatorem rekordu i wersją formatu. Dobór formatu i zakresu powiązania wymaga przeglądu. [W3C — Web Cryptography](https://www.w3.org/TR/webcrypto/)
2. Osobna fraza odblokowująca chroni zaszyfrowaną kopię klucza danych. Kandydatem do wyprowadzania klucza z frazy jest Argon2id, z losową solą i parametrami dobranymi na rzeczywistym iPhonie. Ograniczenie prób w interfejsie służy lokalnej blokadzie. Odporność na zgadywanie przy wycieku zaszyfrowanej kopii zależy od frazy i parametrów funkcji. [RFC 9106 — Argon2](https://www.rfc-editor.org/rfc/rfc9106.html)
3. Niezależna zaszyfrowana kopia klucza danych umożliwia odzyskanie za pomocą losowego kodu o wysokiej entropii. Użytkownik przechowuje kod poza aplikacją. Po zalogowaniu na nowym urządzeniu fraza lub kod odblokowują dane lokalnie. Odzyskanie dostępu do Google przywraca dostęp do szyfrogramów. Odczyt nadal wymaga frazy, kodu albo innej wcześniej zatwierdzonej metody odzyskania.
4. Klucz w jawnej postaci pozostaje w pamięci odblokowanej sesji. Blokada ręczna, wylogowanie i upływ czasu sesji zamykają widoki prywatne i usuwają dostępne odwołania do klucza oraz odszyfrowanych danych. Powrót z tła, czas bezczynności, zmiana frazy, rotacja klucza oraz skutki utraty wszystkich metod odzyskiwania wymagają testów i ustaleń.

## Powierzchnie ujawnienia

- Trwała pamięć aplikacji, service worker i kolejka synchronizacji mogą zachowywać zasoby aplikacji, zwykłe dane zgodnie z polityką urządzenia oraz wyłącznie zaszyfrowane rekordy prywatne. Logi, telemetria, adresy URL, komunikaty błędów i powiadomienia pozostają neutralne.
- Neutralne identyfikatory i wspólna kolekcja prywatna ograniczają ujawnianie przez ścieżki. Serwer wciąż może zaobserwować istnienie zaszyfrowanych dokumentów oraz ich wielkość i częstotliwość aktualizacji.
- Powiadomienie wykorzystuje neutralną treść i sygnały niezależne od prywatnych wpisów. Dostęp do tokenów push ma wyłącznie mechanizm ich rejestracji i wysyłki.
- Odszyfrowane dane są dostępne kodowi odblokowanej aplikacji. Złośliwy skrypt, podmieniona wersja PWA lub przejęte urządzenie mogą uzyskać do nich dostęp. Proponowane zabezpieczenia obejmują HTTPS, restrykcyjną politykę CSP, ograniczenie zewnętrznych skryptów, kontrolę zależności i dostępów do wdrożeń oraz test podglądu PWA przy przejściu w tło.
- Pełna specyfikacja i instrukcja bieżącego środowiska są lokalne. Publiczna dokumentacja opisuje strefę prywatną ogólnie. Dane rzeczywiste, identyfikatory kont, sekrety i kody odzyskiwania pozostają poza publicznymi plikami i zasobami hostingu. Zmiana zawartości bieżącej gałęzi pozostawia wcześniejsze wersje w historii Git.

## Kopia i odzyskiwanie

Odzyskanie klucza i odtworzenie usuniętych danych to odrębne operacje. Propozycja kopii zakłada ręczny eksport do zaszyfrowanego pliku z wersją formatu, weryfikację importu przed zapisem i test odtworzenia na drugim urządzeniu. Szczegóły obejmują m.in. zachowanie ochrony prywatnych rekordów i wymagania dostępu do kopii po utracie urządzenia. [ADR-006](adr.md#adr-006)
