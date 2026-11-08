# Lifesaver — bezpieczeństwo i prywatność

**Status:** zakres E2EE i kontrola dostępu do danych zwykłych są zaakceptowane. Mechanizmy opisane poniżej stanowią projekt do weryfikacji. [ADR-001](adr.md#adr-001) · [ADR-003](adr.md#adr-003)

## Klasy danych i granice zaufania

| Klasa | Zakres | Ochrona |
| --- | --- | --- |
| Zwykłe dane konta | Zwykłe nawyki, sen, pomiary, cele tygodniowe i ogólne ustawienia | Logowanie Google przez Firebase Authentication i reguły Firestore. Usługa przetwarza treść po stronie serwera |
| Dane w E2EE | Dwa prywatne nawyki oraz prywatny licznik, wraz z ich nazwami, regułami, datami, stanami i pomiarami | Szyfrowanie na urządzeniu przed zapisem. Odczyt treści po osobnym odblokowaniu |
| Metadane i obsługa usług | UID właściciela, liczba i rozmiar szyfrogramów, techniczne czasy zapisu, subskrypcje push | Minimalizacja pól, uprawnień i czasu przechowywania |

Pełny wynik i kompletność mogą ujawnić stan prywatnych wpisów przez porównanie z widokiem zwykłym. Propozycja przewiduje obliczanie pełnego wyniku tylko po odblokowaniu, w strefie prywatnej. Do ustalenia pozostaje, czy widok ogólny pokazuje wyraźnie opisaną statystykę zwykłych nawyków, czy pomija wspólny wynik.

## Dostęp do Firestore

- Proponowane reguły zaczynają od odmowy dostępu. Odczyt i zapis wymagają uwierzytelnienia, zgodności UID ze ścieżką właściciela oraz dopuszczenia konta właściciela aplikacji. Sposób dopuszczenia jednego konta wymaga wyboru.
- Reguły walidują typy, dozwolone pola i rozmiar dokumentów. W prywatnej kolekcji dopuszczają wyłącznie kopertę techniczną. Treść wewnątrz szyfrogramu waliduje klient po odszyfrowaniu.
- Konta i procesy administracyjne wymagają odrębnych uprawnień IAM. Harmonogram przypomnień powinien mieć dostęp wyłącznie do stanu wysyłki i subskrypcji. Testy reguł obejmują niezalogowanego użytkownika, obce konto, próby zapisu jawnych pól i prawidłowe operacje właściciela. [Firebase — reguły i uwierzytelnianie](https://firebase.google.com/docs/rules/rules-and-auth) · [Firestore — kontrola dostępu](https://firebase.google.com/docs/firestore/security/get-started)

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
- Dokumentacja prywatna może opisywać funkcje produktu. W repozytorium, przykładach i historii zmian przechowujemy wyłącznie opisy i dane syntetyczne. Dane rzeczywiste, identyfikatory kont, sekrety i kody odzyskiwania pozostają poza repozytorium. Przed ewentualnym upublicznieniem trzeba przejrzeć również historię Git i usunąć z materiałów publicznych prywatne szczegóły konfiguracji oraz opisy ujawniające osobisty kontekst. To samo dotyczy zasobów udostępnianych przez publiczny hosting.

## Kopia i odzyskiwanie

Odzyskanie klucza i odtworzenie usuniętych danych to odrębne operacje. Propozycja kopii zakłada ręczny eksport do zaszyfrowanego pliku z wersją formatu, weryfikację importu przed zapisem i test odtworzenia na drugim urządzeniu. Szczegóły obejmują m.in. zachowanie ochrony prywatnych rekordów i wymagania dostępu do kopii po utracie urządzenia. [ADR-006](adr.md#adr-006)
