# ADR-010: Dokumentacja projektu w Docusaurusie na GitHub Pages

- **Status:** Proponowana
- **Data:** 2026-11-20
- **Autorzy:** Maciej Bułhak (propozycja do dyskusji w zespole)

---

## 1. Kontekst

- WDZPD: na koniec semestru cała dokumentacja (produktowa, techniczna, użytkownika, API) ma być w katalogu `zaliczenie/` i opublikowana jako strona statyczna, np. na GitHub Pages.
- Prowadzący dopuszcza generatory: retype, docsify, fumadocs, docusaurus, nextra, vitepress, astro starlight. Inne tylko po jego akceptacji.
- Dokumentacja API ma zawierać wyrenderowaną specyfikację OpenAPI. Dopuszczone renderery: Stoplight Elements, Scalar, Redoc, Swagger UI.
- Mile widziane diagramy, np. ERD i UML w mermaid.js.
- Backend ma też własną interaktywną dokumentację pod `/docs` (założenia 3.1).

---

## 2. Rozważane opcje

### Opcja 1: Docusaurus

- **Zalety:**
  - [+] Oparty na React, czyli na tej samej technologii co frontend ([ADR-003](003-frontend-react-typescript.md)).
  - [+] Diagramy mermaid przez oficjalny motyw `@docusaurus/theme-mermaid`, specyfikacja OpenAPI przez wtyczkę z Redoc.
  - [+] Gotowa instrukcja publikacji na GitHub Pages przez GitHub Actions.
- **Wady:**
  - [-] Cięższy od pozostałych opcji; konfiguracja w JavaScripcie.

### Opcja 2: VitePress

- **Zalety:**
  - [+] Bardzo szybki i prosty, nastawiony na Markdown.
- **Wady:**
  - [-] Oparty na Vue; mermaid i OpenAPI tylko przez wtyczki społeczności.

### Opcja 3: Astro Starlight

- **Zalety:**
  - [+] Szybki, dobra dostępność, wtyczka do OpenAPI.
- **Wady:**
  - [-] Kolejny ekosystem (Astro) do poznania.

### Opcja 4: docsify

- **Zalety:**
  - [+] Bez kroku budowania: sam Markdown i jeden plik HTML.
- **Wady:**
  - [-] Treść renderowana w przeglądarce, mniej funkcji (wersjonowanie, OpenAPI).

---

## 3. Decyzja

**Wybieramy Docusaurus w katalogu `zaliczenie/`, publikowany przez GitHub Actions na GitHub Pages. Specyfikację OpenAPI eksportujemy z backendu w CI i renderujemy na stronie przez Redoc. W samym backendzie pod `/docs` udostępniamy Swagger UI z drf-spectacular.**

---

## 4. Uzasadnienie

Docusaurus jest na liście prowadzącego, ma oficjalne wsparcie mermaid i dobrą wtyczkę do OpenAPI, a znajomość Reacta z frontendu wystarczy do ewentualnych dostosowań. Swagger UI w backendzie (także z listy prowadzącego) daje drf-spectacular bez dodatkowej pracy ([ADR-002](002-backend-django-rest-framework.md)); jest zgodny z przykładem z założeń (3.1) i pozwala wywoływać endpointy z przeglądarki.

---

## 5. Konsekwencje

### Pozytywne
- Dokumentacja API na stronie zawsze odpowiada aktualnemu kodowi, bo specyfikacja jest eksportowana w CI.
- Jedno miejsce na wszystkie artefakty WDZPD: PRD, persony, user stories, backlog, sprinty, retrospektywy, ADR-y, event storming.

### Negatywne
- Strona dokumentacji to kolejny projekt npm do aktualizowania.

### Neutralne (zadania)
- Przy zakładaniu strony przenosimy obecny katalog `docs/` (założenia, ADR-y, event storming) do `zaliczenie/docs/`, zgodnie ze strukturą z szablonu laboratorium.
- Włączenie GitHub Pages w ustawieniach repozytorium (wymaga uprawnień właściciela).
