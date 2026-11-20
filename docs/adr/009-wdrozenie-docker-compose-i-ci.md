# ADR-009: Wdrożenie w Docker Compose na VPS, CI w GitHub Actions

- **Status:** Proponowana
- **Data:** 2026-11-20
- **Autorzy:** Maciej Bułhak (propozycja do dyskusji w zespole)

---

## 1. Kontekst

- Wdrożenie w kontenerach (Docker, docker-compose), osobno frontend, backend i baza danych; konfiguracja przez zmienne środowiskowe (3.10).
- CI/CD z automatycznym uruchamianiem testów i lintera (3.10); testy jednostkowe, integracyjne i E2E kluczowych scenariuszy.
- PPSI2: projekt skonteneryzowany, z testami automatycznymi; na zaliczeniu prezentujemy działającą i wdrożoną aplikację.
- HTTPS (3.1), endpoint `/health` (3.7), cykliczne kopie zapasowe z procedurą odtworzenia (3.6).
- Frontend i backend w jednej domenie, żeby działało ciasteczko z refresh tokenem ([ADR-005](005-uwierzytelnianie-jwt-i-google.md)).
- Brak budżetu.

---

## 2. Rozważane opcje

### Opcja 1: Jeden VPS z Docker Compose i Caddy jako reverse proxy z automatycznym HTTPS

Serwer np. z kredytów dla studentów (GitHub Student Developer Pack).

- **Zalety:**
  - [+] Ten sam `docker-compose.yml` co lokalnie, więc najmniej różnic między środowiskami.
  - [+] Pełna kontrola: PostGIS, wolumeny na pliki PDF, cykliczne kopie bazy.
  - [+] Caddy sam pobiera i odnawia certyfikaty Let's Encrypt.
- **Wady:**
  - [-] Utrzymanie serwera (aktualizacje, firewall, kopie) spada na zespół.
  - [-] Potrzebna domena z subdomenami `app.` i `api.`.

### Opcja 2: Platforma PaaS (np. Render, Railway, Fly.io)

- **Zalety:**
  - [+] Wdrożenie prosto z repozytorium, bez administrowania serwerem.
- **Wady:**
  - [-] Darmowe plany usypiają usługi albo mają limity; PostGIS i trwałe wolumeny nie wszędzie są dostępne.
  - [-] Konfiguracja inna niż lokalny docker-compose.

### Opcja 3: Azure for Students (kontenery w App Service, Azure Database for PostgreSQL)

- **Zalety:**
  - [+] Kredyt dla studentów; zarządzana baza z kopiami zapasowymi i rozszerzeniem PostGIS.
- **Wady:**
  - [-] Wiele usług do skonfigurowania w portalu, kredyt jest ograniczony i łatwo go przekroczyć przy złej konfiguracji.

---

## 3. Decyzja

**Wybieramy VPS z Docker Compose (kontenery: `frontend` z nginx, `backend`, `worker` i `beat` z Celery, `redis`, `db` z PostGIS, `caddy`) oraz CI w GitHub Actions z obrazami w GitHub Container Registry.**

Potok CI dla każdego pull requesta:
- backend: Ruff (lint i formatowanie), testy jednostkowe i integracyjne w pytest-django na bazie PostGIS uruchomionej jako usługa w GitHub Actions;
- frontend: ESLint, sprawdzenie typów, testy (Vitest), budowanie aplikacji i Storybooka;
- testy E2E kluczowych scenariuszy (Playwright) na złożonym środowisku z docker-compose.

Po scaleniu do `main`: budowanie obrazów, publikacja w GHCR i wdrożenie na serwer.

---

## 4. Uzasadnienie

Lokalnie i tak uruchamiamy system przez docker-compose, więc wdrożenie tego samego pliku na serwerze to najkrótsza droga do spełnienia 3.10 i do prezentacji na PPSI2. PaaS oszczędza administrację, ale darmowe plany usypiają aplikację i różnie wspierają PostGIS. Azure daje zarządzaną bazę, ale to dużo konfiguracji i ryzyko wyczerpania kredytu.

---

## 5. Konsekwencje

### Pozytywne
- Jedno źródło konfiguracji środowisk: `docker-compose.yml` z plikiem `.env`.
- Każdy pull request jest sprawdzony testami i linterem przed scaleniem.

### Negatywne
- Ktoś musi utrzymywać serwer. Mitygacja: przypisana rola devopsa (PPSI2 rozlicza z wdrożenia osobę o tej roli, a bez niej cały zespół).

### Neutralne (zadania)
- Ochrona gałęzi `main`: scalanie tylko przez pull request z zielonym CI (checklista startu projektu z WDZPD).
- Kopie: nocny `pg_dump` i kopia wolumenu z PDF poza serwer; procedura odtworzenia w dokumentacji technicznej.
- Do sprawdzenia przed decyzją: aktualne warunki ofert studenckich (VPS, domena) i dostawcy SMTP ([ADR-007](007-zadania-w-tle-celery-redis.md)).
