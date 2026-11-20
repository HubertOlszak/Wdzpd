# ADR-005: Uwierzytelnianie: krótki JWT z refresh tokenem i logowanie przez Google (OIDC)

- **Status:** Proponowana
- **Data:** 2026-11-20
- **Autorzy:** Maciej Bułhak (propozycja do dyskusji w zespole)

---

## 1. Kontekst

- Rejestracja e-mailem i hasłem z aktywacją linkiem, reset hasła przez e-mail, edycja i usunięcie konta (2.1).
- Rejestracja i logowanie przez zewnętrznego dostawcę OAuth 2.0 / OIDC, np. Google (2.1); wymóg tematu z PPSI2.
- REST API ma być bezstanowe, a frontend i backend wdrażane osobno (3.1).
- Hasła haszowane algorytmem adaptacyjnym (bcrypt lub Argon2), z wymaganą minimalną siłą (3.3).
- Role: wolontariusz, organizator (członek konkretnej organizacji), administrator; kontrola własności zasobów, np. organizacja edytuje tylko własne akcje (3.3).
- Zablokowany użytkownik albo organizacja muszą stracić dostęp (2.3, 2.11).
- Backend w Django z Django REST Framework ([ADR-002](002-backend-django-rest-framework.md)).

---

## 2. Rozważane opcje

### Opcja 1: JWT z krótkim czasem życia i refresh token w ciasteczku HttpOnly; konta i Google przez django-allauth

Access token JWT (djangorestframework-simplejwt) żyje około 15 minut. Refresh token jest rotowany przy każdym użyciu, a zużyte trafiają na czarną listę. Refresh token leży w ciasteczku HttpOnly. Rejestrację, aktywację e-mailem, reset hasła i logowanie przez Google (authorization code flow po stronie backendu) obsługuje django-allauth, wystawione przez API.

- **Zalety:**
  - [+] API pozostaje bezstanowe, zgodnie z 3.1.
  - [+] allauth i simplejwt to sprawdzone biblioteki; nie piszemy własnej kryptografii ani obsługi OAuth.
  - [+] Refresh token w ciasteczku HttpOnly jest niedostępny dla JavaScriptu, więc trudniej go wykraść przez XSS.
  - [+] Argon2 to wbudowany hasher Django (wystarczy go ustawić jako pierwszy), a siłę hasła sprawdzają wbudowane walidatory.
- **Wady:**
  - [-] Access tokenu nie da się unieważnić przed wygaśnięciem, stąd krótki czas życia.
  - [-] allauth trzeba połączyć z JWT: przez tryb headless allauth albo przez dj-rest-auth. Wybieramy na początku sprintu 1, po próbie obu.

### Opcja 2: Sesje Django w ciasteczku

- **Zalety:**
  - [+] Najprostsze w Django (tak działał projekt z PPSI1), natychmiastowe wylogowanie i blokada.
- **Wady:**
  - [-] Stan po stronie serwera, niezgodny z bezstanowym API z założeń.
  - [-] Wymaga ochrony CSRF i dopilnowania ciasteczek między domeną frontendu a backendu.

### Opcja 3: Zewnętrzny serwer tożsamości (Keycloak albo usługa typu Auth0)

- **Zalety:**
  - [+] Gotowe logowanie, rejestracja, OAuth i panel zarządzania kontami.
- **Wady:**
  - [-] Dodatkowy kontener do skonfigurowania (Keycloak) albo zależność od dostawcy z limitami darmowego planu.
  - [-] Trudniej dopasować ekrany logowania do design systemu z PIG.

---

## 3. Decyzja

**Wybieramy opcję 1: konta i logowanie przez Google w django-allauth, hasła w Argon2, krótki JWT jako access token i rotowany refresh token w ciasteczku HttpOnly (djangorestframework-simplejwt z czarną listą).**

---

## 4. Uzasadnienie

Opcja 1 jako jedyna spełnia jednocześnie bezstanowe API i osobne wdrożenia frontendu i backendu, a najtrudniejsze części (aktywacja e-mailem, reset hasła, OAuth z Google, haszowanie) dają gotowe biblioteki. Sesje są w Django najprostsze, ale łamią założenie 3.1. Zewnętrzny serwer tożsamości to sporo dodatkowej konfiguracji przy jednym dostawcy logowania.

---

## 5. Konsekwencje

### Pozytywne
- Jeden mechanizm autoryzacji dla kont e-mailowych i kont z Google.
- Blokada użytkownika działa najpóźniej po wygaśnięciu access tokenu (około 15 minut); jego refresh tokeny trafiają od razu na czarną listę.

### Negatywne
- Ciasteczko z refresh tokenem wymaga, żeby frontend i backend były w tej samej domenie (np. subdomeny `app.` i `api.`). Inaczej przeglądarki mogą je blokować jako ciasteczko stron trzecich. Uwzględniamy to w [ADR-009](009-wdrozenie-docker-compose-i-ci.md).

### Neutralne (zadania)
- Rejestracja aplikacji w Google Cloud Console: klient OAuth i adresy przekierowań dla środowiska lokalnego i produkcji. Sekrety tylko w zmiennych środowiskowych, nigdy w repozytorium.
- Role globalne (użytkownik, administrator) plus członkostwo w organizacji z rolą organizatora, sprawdzane klasami uprawnień DRF.
- Do ustalenia: łączenie konta e-mailowego z kontem Google o tym samym, zweryfikowanym adresie.
