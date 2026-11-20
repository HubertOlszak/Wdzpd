# ADR-008: Potwierdzenia udziału jako PDF generowany przez WeasyPrint z szablonu HTML

- **Status:** Proponowana
- **Data:** 2026-11-20
- **Autorzy:** Maciej Bułhak (propozycja do dyskusji w zespole)

---

## 1. Kontekst

- Organizator wystawia elektroniczne potwierdzenie udziału jako PDF A4 z polskimi znakami. Zawiera dane wolontariusza, organizacji i akcji, termin, liczbę godzin, datę wystawienia, unikalny numer oraz kod QR lub link weryfikacyjny (2.7, 3.8).
- Potwierdzenie jest niezmienne po wystawieniu; korekta tworzy nową wersję i unieważnia poprzednią (3.8).
- Dostęp do PDF mają tylko właściciel, organizacja wystawiająca i administrator. Publiczna weryfikacja po numerze lub kodzie ujawnia tylko minimum danych (2.7, 3.3).
- Generowanie ma działać asynchronicznie (3.5, [ADR-007](007-zadania-w-tle-celery-redis.md)).
- Backend w Django ([ADR-002](002-backend-django-rest-framework.md)).

---

## 2. Rozważane opcje

### Opcja 1: WeasyPrint (szablon Django w HTML i CSS zamieniany na PDF) i segno do kodu QR

- **Zalety:**
  - [+] Układ dokumentu w szablonie HTML i CSS, czyli w tym, co znamy z szablonów Django.
  - [+] Obsługa CSS dla druku: format A4, marginesy, osadzone czcionki z polskimi znakami.
  - [+] Kod QR jako obraz SVG wstawiony do szablonu.
- **Wady:**
  - [-] Wymaga bibliotek systemowych (Pango) w obrazie Dockera.
  - [-] Obsługuje tylko część CSS (np. bez JavaScriptu), więc szablon musi być prosty.

### Opcja 2: ReportLab (układ dokumentu w kodzie Pythona)

- **Zalety:**
  - [+] Pełna kontrola nad PDF, wbudowany generator kodów QR.
- **Wady:**
  - [-] Układ pisany w kodzie, więc zmiany wyglądu są pracochłonne.
  - [-] Polskie znaki wymagają ręcznej rejestracji czcionki TTF.

### Opcja 3: HTML zamieniany na PDF przez przeglądarkę bezgłową (Playwright albo Gotenberg)

- **Zalety:**
  - [+] Pełne wsparcie HTML i CSS.
- **Wady:**
  - [-] Chromium w obrazie backendu albo dodatkowa usługa (Gotenberg): duży obraz i więcej pamięci.

---

## 3. Decyzja

**Wybieramy WeasyPrint z szablonem Django i segno do kodów QR. PDF powstaje raz, w zadaniu w tle po wystawieniu potwierdzenia, i jest przechowywany jako plik wraz z sumą kontrolną SHA-256 zapisaną w bazie.**

---

## 4. Uzasadnienie

WeasyPrint pozwala zbudować potwierdzenie jak zwykłą stronę w szablonie Django i ostylować je kolorami i czcionkami z design systemu, a koszt to kilka pakietów systemowych w obrazie. ReportLab wymaga układania dokumentu w kodzie. Przeglądarka bezgłowa daje więcej możliwości, ale mocno powiększa wdrożenie.

---

## 5. Konsekwencje

### Pozytywne
- Raz wygenerowany plik się nie zmienia, a suma kontrolna pozwala to sprawdzić (3.8).
- Generowanie nie spowalnia odpowiedzi API.

### Negatywne
- Pliki PDF trzeba objąć kopiami zapasowymi razem z bazą ([ADR-009](009-wdrozenie-docker-compose-i-ci.md)).
- Obraz backendu rośnie o biblioteki systemowe WeasyPrint (obok bibliotek GeoDjango).

### Neutralne (zadania)
- Numer potwierdzenia czytelny dla człowieka (np. `WDZ-2027-000123`), a w linku weryfikacyjnym i kodzie QR dodatkowo losowy token, żeby nie dało się przeglądać cudzych potwierdzeń przez zgadywanie kolejnych numerów.
- Do ustalenia: jakie dane pokazuje publiczna weryfikacja (propozycja: inicjały wolontariusza, organizacja, akcja, data, liczba godzin, status „ważne” albo „unieważnione”).
