# ADR-001: Osobny frontend SPA i backend REST jako modularny monolit, w jednym repozytorium

- **Status:** Proponowana
- **Data:** 2026-11-20
- **Autorzy:** Maciej Bułhak (propozycja do dyskusji w zespole)

---

## 1. Kontekst

- Wymagania tematu z PPSI2: backend i frontend odseparowane, komunikacja przez REST API, backend udokumentowany w OpenAPI. W [założeniach](../za%C5%82o%C5%BCenia.md) (3.1): osobne wdrożenia, JSON przez HTTPS, wersjonowanie `/api/v1`, dokumentacja pod `/docs`.
- PIG: biblioteka komponentów musi być w technologii frontendu z PPSI2 i faktycznie użyta w aplikacji, a w `README.md` repozytorium ma być link do pliku Figma.
- WDZPD: wszystkie dokumenty w katalogu `zaliczenie/`, opublikowane jako strona statyczna.
- Zapis na akcję i awans z listy rezerwowej muszą być transakcyjne i odporne na wyścigi (3.6).
- Zespół 4-osobowy, jeden semestr, praca głównie zdalna. Docelowe obciążenie jest małe: 100 równoległych użytkowników (3.5).

---

## 2. Rozważane opcje

### Opcja 1: Modularny monolit i SPA w jednym repozytorium

Jeden backend podzielony na moduły według kontekstów z event stormingu, osobna aplikacja SPA. Katalogi `backend/`, `frontend/` i `zaliczenie/` w repozytorium grupy.

- **Zalety:**
  - [+] Jedna baza danych, więc zapis na akcję i awans z listy rezerwowej mieszczą się w jednej transakcji.
  - [+] Jedno wdrożenie backendu i prosty start lokalny (`docker compose up`).
  - [+] Moduły pozwalają podzielić pracę w zespole bez kosztu usług rozproszonych.
  - [+] Jedno repozytorium do zgłoszenia na trzech przedmiotach, wspólne issues, tablica i historia zmian.
- **Wady:**
  - [-] Granic modułów pilnuje tylko dyscyplina zespołu i przeglądy kodu.
  - [-] CI musi rozróżniać zmiany frontendu i backendu (filtry ścieżek w GitHub Actions).

### Opcja 2: Mikroserwisy i SPA

Osobne usługi, np. konta, akcje, zapisy, powiadomienia, każda z własną bazą.

- **Zalety:**
  - [+] Niezależne wdrażanie i skalowanie usług.
- **Wady:**
  - [-] Zapis i awans z listy rezerwowej wymagają transakcji rozproszonych (saga, outbox).
  - [-] Wiele kontenerów, baz i konfiguracji do utrzymania przez cztery osoby w jednym semestrze.
  - [-] Przy zakładanym ruchu nie ma potrzeby niezależnego skalowania.

### Opcja 3: Aplikacja full-stack z renderowaniem po stronie serwera

Np. Next.js albo Laravel z szablonami Blade.

- **Zalety:**
  - [+] Najmniej kodu łączącego frontend z backendem.
- **Wady:**
  - [-] Nie spełnia wymagania PPSI2: frontend i backend mają być odseparowane i komunikować się przez REST API.

---

## 3. Decyzja

**Wybieramy modularny monolit (backend REST) i osobną aplikację SPA, oba w jednym repozytorium grupy.**

---

## 4. Uzasadnienie

Opcja 3 odpada przez wymagania tematu. Mikroserwisy rozwiązują problemy skali i wielu zespołów, których nie mamy, a dokładają transakcje rozproszone dokładnie tam, gdzie założenia wymagają spójności: przy limicie miejsc i liście rezerwowej. Modularny monolit spełnia wymagania (osobne wdrożenie frontendu i backendu, REST, OpenAPI) najmniejszym kosztem, a podział na moduły zostawia porządek w kodzie i podział pracy. Jedno repozytorium upraszcza zgłaszanie projektu na PPSI2, PIG i WDZPD.

---

## 5. Konsekwencje

### Pozytywne
- Spójność danych przy zapisach zapewnia zwykła transakcja bazy danych.
- Cały system uruchamia się lokalnie jednym poleceniem.

### Negatywne
- Zmiany w jednym module mogą przypadkiem zależeć od innego. Mitygacja: moduły komunikują się przez interfejsy i zdarzenia w procesie, a nie przez cudze tabele; przegląd kodu w PR pilnuje granic.

### Neutralne (zadania)
- Moduły backendu odpowiadają kontekstom z [event stormingu](../event-storming.md#konteksty-ograniczone): konta i profil, organizacje, akcje, zapisy, realizacja i potwierdzenia, powiadomienia, rekomendacje i statystyki, administracja.
- Struktura repozytorium: `backend/`, `frontend/`, `zaliczenie/` (dokumentacja WDZPD, [ADR-010](010-dokumentacja-docusaurus.md)), `docker-compose.yml` w katalogu głównym.
