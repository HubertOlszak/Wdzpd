# ADR-003: Frontend w React z TypeScriptem i Vite

- **Status:** Proponowana
- **Data:** 2026-11-20
- **Autorzy:** Maciej Bułhak (propozycja do dyskusji w zespole)

---

## 1. Kontekst

- Frontend to responsywna aplikacja SPA po polsku (3.1, 3.9). PIG wymaga podejścia mobile-first i trzech szerokości: od 360 px, od 768 px i od 1280 px.
- PIG: biblioteka co najmniej 12 komponentów w technologii frontendu, faktycznie użytych w aplikacji. Prezentacja w Storybooku jest premiowana. Tokeny z Figmy mają mieć w kodzie te same nazwy i wartości.
- Wyszukiwarka z listą i mapą zsynchronizowanymi z filtrami oraz geolokalizacją przeglądarki (2.5).
- Kontrakt z backendem opisuje specyfikacja OpenAPI ([ADR-002](002-backend-django-rest-framework.md)).

---

## 2. Rozważane opcje

### Opcja 1: React + TypeScript + Vite

- **Zalety:**
  - [+] Największy ekosystem: react-leaflet (mapa), TanStack Query (dane z API), React Router, React Hook Form.
  - [+] Pełne wsparcie Storybooka i najwięcej przykładów bibliotek komponentów.
  - [+] Typy klienta API można generować ze specyfikacji OpenAPI (np. `openapi-typescript`).
  - [+] Wiedza przenosi się na React Native, czyli na aplikacje mobilne z naszej specjalności.
- **Wady:**
  - [-] Routing, pobieranie danych i formularze trzeba dobrać z osobnych bibliotek.
  - [-] JSX i hooki są nowe dla osób, które nie pisały w React.

### Opcja 2: Vue 3 + TypeScript + Vite

- **Zalety:**
  - [+] Łagodniejsza krzywa nauki, szablony bliskie HTML.
  - [+] Storybook i Leaflet (vue-leaflet) też działają.
- **Wady:**
  - [-] Mniejszy ekosystem gotowych komponentów i przykładów.

### Opcja 3: Angular

- **Zalety:**
  - [+] Kompletny framework: routing, formularze i HTTP bez wybierania bibliotek.
- **Wady:**
  - [-] Najwięcej pojęć do nauki (moduły, RxJS, wstrzykiwanie zależności) w jednym semestrze.

---

## 3. Decyzja

**Wybieramy React z TypeScriptem, budowany przez Vite. Biblioteka komponentów dla PIG powstaje w tym samym projekcie i jest prezentowana w Storybooku.**

---

## 4. Uzasadnienie

React ma najlepiej wspierane wszystkie elementy, których wymagają założenia i PIG: mapę z markerami, Storybooka, generowanie typów z OpenAPI. TypeScript wyłapuje niezgodności z API już przy kompilacji. Vue byłby łatwiejszy na start, ale ma mniej gotowych rozwiązań. Angular to za dużo nauki na jeden semestr.

---

## 5. Konsekwencje

### Pozytywne
- Jedna baza komponentów dla Storybooka (PIG) i aplikacji (PPSI2), więc spełniamy wymaganie, że biblioteka jest faktycznie używana.
- Typy z OpenAPI ograniczają błędy na styku frontendu i backendu.

### Negatywne
- Osoby bez doświadczenia z Reactem potrzebują czasu na naukę. Mitygacja: wspólne konwencje (struktura katalogów, sposób pobierania danych) ustalone w pierwszym sprincie i opisane w `CONTRIBUTING.md`.

### Neutralne (zadania)
- Tokeny z Figma Variables (kolory, typografia, odstępy, zaokrąglenia, cienie) trafiają do kodu jako zmienne CSS o tych samych nazwach.
- Do ustalenia: sposób stylowania (CSS Modules albo Tailwind). Według przewodnika ADR to decyzja łatwo odwracalna, więc nie wymaga osobnego ADR-a.
