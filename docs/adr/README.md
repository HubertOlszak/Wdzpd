# Decyzje architektoniczne (ADR)

Rejestr decyzji technicznych projektu. Format jest zgodny z szablonem z laboratorium WDZPD: kontekst, rozważane opcje, decyzja, uzasadnienie, konsekwencje.

Wszystkie ADR-y poniżej mają status **Proponowana**, czyli są propozycją do omówienia w zespole. Po akceptacji zmieniamy status na „Zaakceptowana” i wpisujemy autorów decyzji. Zaakceptowanego ADR-a już nie edytujemy. Jeśli decyzja się zmieni, piszemy nowy ADR, który zastępuje stary, a stary oznaczamy jako „Zastąpiona”.

| Nr | Decyzja | Status |
|---|---|---|
| [ADR-001](001-architektura-spa-i-rest-api.md) | Osobny frontend SPA i backend REST jako modularny monolit, w jednym repozytorium | Proponowana |
| [ADR-002](002-backend-django-rest-framework.md) | Backend w Django z Django REST Framework | Proponowana |
| [ADR-003](003-frontend-react-typescript.md) | Frontend w React z TypeScriptem i Vite | Proponowana |
| [ADR-004](004-baza-postgresql-postgis.md) | PostgreSQL z PostGIS jako baza danych | Proponowana |
| [ADR-005](005-uwierzytelnianie-jwt-i-google.md) | Uwierzytelnianie: krótki JWT z refresh tokenem i logowanie przez Google (OIDC) | Proponowana |
| [ADR-006](006-mapa-leaflet-osm.md) | Mapa w Leaflet z kafelkami OpenStreetMap, geokodowanie przez Nominatim | Proponowana |
| [ADR-007](007-zadania-w-tle-celery-redis.md) | Zadania w tle w Celery z Redisem, powiadomienia w aplikacji i e-mailem | Proponowana |
| [ADR-008](008-pdf-potwierdzen-weasyprint.md) | Potwierdzenia udziału jako PDF generowany przez WeasyPrint z szablonu HTML | Proponowana |
| [ADR-009](009-wdrozenie-docker-compose-i-ci.md) | Wdrożenie w Docker Compose na VPS, CI w GitHub Actions | Proponowana |
| [ADR-010](010-dokumentacja-docusaurus.md) | Dokumentacja projektu w Docusaurusie na GitHub Pages | Proponowana |

## Zależności między decyzjami

- ADR-002 (backend) jest decyzją kluczową. Proponujemy Django, bo trzy osoby z zespołu znają je z projektu na PPSI1. ADR-005, ADR-007 i ADR-008 dobierają biblioteki pod Django. Jeśli zespół wybierze inny backend, te trzy ADR-y trzeba przejrzeć i zmienić biblioteki; same decyzje (JWT, kolejka zadań, PDF generowany na serwerze) zostają.
- ADR-003 (frontend) wyznacza też technologię biblioteki komponentów na PIG.
- ADR-001, ADR-004 i ADR-006 wynikają głównie z wymagań tematu i nie zależą od wyboru języka.

## Do ustalenia na zajęciach

1. Potwierdzenie Django jako backendu (ADR-002) i wybór frameworka frontendu (ADR-003), także z perspektywy czwartej osoby w zespole.
2. Kto bierze rolę devopsa (PPSI2 wymaga jej przypisania albo cały zespół odpowiada za wdrożenie): od tego zależy ADR-009.
3. Gorące punkty z [event stormingu](../event-storming.md#gorące-punkty), które wpływają na model danych.
