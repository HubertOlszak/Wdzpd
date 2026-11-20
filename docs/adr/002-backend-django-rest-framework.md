# ADR-002: Backend w Django z Django REST Framework

- **Status:** Proponowana
- **Data:** 2026-11-20
- **Autorzy:** Maciej Bułhak (propozycja do dyskusji w zespole)

---

## 1. Kontekst

Backend ma udostępniać REST API i spełnić wymagania z [założeń](../za%C5%82o%C5%BCenia.md):
- specyfikacja OpenAPI 3.x utrzymywana razem z kodem, interaktywna dokumentacja (Swagger UI albo Redoc) pod `/docs` (3.1);
- konta z e-mailem i hasłem (aktywacja, reset hasła), logowanie przez Google (OAuth 2.0 / OIDC), hasła w bcrypt albo Argon2, role i kontrola własności zasobów (2.1, 3.3);
- zapytania przestrzenne w PostgreSQL z PostGIS (3.2, [ADR-004](004-baza-postgresql-postgis.md));
- PDF z polskimi znakami i kodem QR (3.8), zadania w tle (3.5);
- testy jednostkowe i integracyjne, kontener Docker (3.10).

Trzy osoby z zespołu zrobiły na PPSI1 (4. semestr) projekt w Django 4.2 z PostgreSQL i Docker Compose. Tamten projekt nie miał REST API ani Django REST Framework, więc API dla osobnego frontendu będzie dla nas nowe. Doświadczenie czwartej osoby do potwierdzenia.

---

## 2. Rozważane opcje

### Opcja 1: Django 5.2 LTS z Django REST Framework (Python)

- **Zalety:**
  - [+] Framework, baza i Docker Compose znane zespołowi z PPSI1: modele, migracje, panel admina, testy.
  - [+] GeoDjango (`django.contrib.gis`) obsługuje PostGIS wprost: pola punktowe i zapytania „w promieniu” przez ORM.
  - [+] drf-spectacular generuje OpenAPI 3 z kodu i udostępnia Swagger UI oraz Redoc, czyli dokładnie to, czego wymagają założenia.
  - [+] django-allauth daje rejestrację z aktywacją e-mailem, reset hasła i logowanie przez Google; Django ma wbudowany hasher Argon2 i walidatory siły hasła.
  - [+] Wersja LTS ze wsparciem do kwietnia 2028, czyli dłużej niż trwa projekt.
- **Wady:**
  - [-] Django REST Framework (serializery, widoki API, uprawnienia) trzeba poznać od nowa.
  - [-] GeoDjango wymaga bibliotek systemowych (GDAL, GEOS, PROJ) w obrazie Dockera.
  - [-] Zadania w tle wymagają osobnego narzędzia ([ADR-007](007-zadania-w-tle-celery-redis.md)).

### Opcja 2: FastAPI (Python)

- **Zalety:**
  - [+] Zwięzły kod, OpenAPI i Swagger UI generowane z typów (Pydantic), obsługa asynchroniczności.
- **Wady:**
  - [-] Uwierzytelnianie, role, migracje (Alembic), panel administracyjny i obsługę PostGIS (GeoAlchemy2) trzeba składać z osobnych bibliotek.
  - [-] Framework nowy dla zespołu; tracimy doświadczenie z Django.

### Opcja 3: ASP.NET Core (C#)

- **Zalety:**
  - [+] Spójny ekosystem: OpenAPI, Identity, EF Core z PostGIS, biblioteki do PDF i zadań w tle.
  - [+] C# znany z Unity.
- **Wady:**
  - [-] Framework, ORM i sposób pracy nowe dla zespołu; więcej konfiguracji na start.

### Opcja 4: Spring Boot (Java) albo NestJS (TypeScript)

- **Zalety:**
  - [+] Dojrzałe ekosystemy z generowaniem OpenAPI; NestJS dzieli język z frontendem.
- **Wady:**
  - [-] Nowy język albo nowa platforma dla backendu, czyli najwięcej nauki w jednym semestrze.

---

## 3. Decyzja

**Wybieramy Django 5.2 LTS z Django REST Framework (Python 3.12 lub nowszy), GeoDjango do zapytań przestrzennych i drf-spectacular do specyfikacji OpenAPI.**

---

## 4. Uzasadnienie

Django jest jedynym frameworkiem, który zespół już zna, i ma gotowe elementy na prawie wszystkie wymagania niefunkcjonalne: PostGIS przez GeoDjango, OpenAPI ze Swagger UI i Redoc, konta z aktywacją i Google przez allauth, Argon2 w standardzie. Do nauki zostaje Django REST Framework, czyli warstwa API nad znanymi modelami. Pozostałe opcje wymagają nauki całego frameworka. Wydajność Pythona wystarcza przy założonych 100 równoległych użytkownikach i limicie 500 ms (3.5).

---

## 5. Konsekwencje

### Pozytywne
- Szybki start sprintu 1: struktura projektu, modele i Docker Compose jak na PPSI1.
- Specyfikacja OpenAPI powstaje z kodu, więc dokumentacja nie rozjeżdża się z API (3.1).
- Panel admina Django pomaga zespołowi przy testach i danych (słowniki, organizacje), choć nie zastępuje panelu administratora w aplikacji.

### Negatywne
- Na początku trzeba poznać Django REST Framework. Mitygacja: szkielet API (jeden moduł z serializerem, widokiem, uprawnieniami i testem) robi na start jedna osoba, reszta pracuje na gotowym wzorcu.
- Czytelna dokumentacja API wymaga opisów w kodzie (dekoratory `extend_schema`, opisy pól).

### Neutralne (zadania)
- Moduły backendu jako osobne aplikacje Django ([ADR-001](001-architektura-spa-i-rest-api.md)).
- Lintowanie i formatowanie kodu przez Ruff, testy przez pytest-django.
