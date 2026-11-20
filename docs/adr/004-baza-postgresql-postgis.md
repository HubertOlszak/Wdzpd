# ADR-004: PostgreSQL z PostGIS jako baza danych

- **Status:** Proponowana
- **Data:** 2026-11-20
- **Autorzy:** Maciej Bułhak (propozycja do dyskusji w zespole)

---

## 1. Kontekst

- Dane są silnie powiązane: konta, organizacje, akcje, zgłoszenia, obecności, potwierdzenia, słowniki, powiadomienia.
- Wyszukiwanie akcji po odległości od punktu i po obszarze zainteresowania (2.2, 2.5), z indeksami przestrzennymi; wyszukiwanie z mapą w czasie do 1 s (3.2, 3.5).
- Wyszukiwanie po słowach kluczowych w nazwie i opisie akcji (2.5).
- Zapis na akcję i awans z listy rezerwowej muszą być transakcyjne i odporne na wyścigi, żeby nie przekroczyć limitu miejsc (3.6).
- Statystyki organizacji (frekwencja, godziny, unikalni wolontariusze) liczone z wielu tabel (2.10).
- Cykliczne kopie zapasowe z opisaną procedurą odtworzenia (3.6).

---

## 2. Rozważane opcje

### Opcja 1: PostgreSQL z rozszerzeniem PostGIS

- **Zalety:**
  - [+] Typ `geography(Point, 4326)` i `ST_DWithin` z indeksem GiST: promień w metrach bez własnej matematyki.
  - [+] Transakcje ACID i blokady wierszy (`SELECT … FOR UPDATE`) do pilnowania limitu miejsc.
  - [+] Rozszerzenie `pg_trgm` daje szybkie wyszukiwanie fragmentów tekstu z indeksem.
  - [+] Gotowy obraz Dockera (`postgis/postgis`) i natywne wsparcie w GeoDjango ([ADR-002](002-backend-django-rest-framework.md)).
  - [+] PostgreSQL zespół zna z projektu na PPSI1.
- **Wady:**
  - [-] Rozszerzenia (`postgis`, `pg_trgm`, `unaccent`) trzeba włączyć w migracjach.
  - [-] Wbudowane wyszukiwanie pełnotekstowe nie ma słownika polskiego. Odmiany słów obsłużymy przez trigramy, a nie przez lematyzację.

### Opcja 2: MySQL 8 z typami przestrzennymi

- **Zalety:**
  - [+] Popularna baza z typami przestrzennymi i funkcją `ST_Distance_Sphere`.
- **Wady:**
  - [-] Zapytanie o promień nie korzysta wprost z indeksu przestrzennego; trzeba ręcznie zawężać wyniki prostokątem.
  - [-] Uboższy zestaw funkcji przestrzennych niż PostGIS.

### Opcja 3: MongoDB z indeksem 2dsphere

- **Zalety:**
  - [+] Zapytania `$near` i `$geoWithin` z indeksem.
  - [+] Elastyczny schemat.
- **Wady:**
  - [-] Zgłoszenia, obecności, potwierdzenia i statystyki to dane relacyjne, wymagające wielu złączeń.
  - [-] Pilnowanie limitu miejsc wymaga transakcji wielodokumentowych albo starannego modelowania dokumentów.

---

## 3. Decyzja

**Wybieramy PostgreSQL z PostGIS. Współrzędne zapisujemy jako `geography(Point, 4326)`, a wyszukiwanie po słowach kluczowych opieramy na `pg_trgm`.**

---

## 4. Uzasadnienie

PostGIS jest standardem do zapytań „w promieniu X km” i robi je z indeksem, czego wymaga limit czasu wyszukiwania z mapą. Relacyjny model i transakcje z blokadą wiersza rozwiązują najtrudniejszą regułę domeny, czyli limit miejsc i kolejkę rezerwową. MySQL wymagałby obejść przy zapytaniach przestrzennych, a MongoDB komplikuje transakcje i statystyki.

---

## 5. Konsekwencje

### Pozytywne
- Wyszukiwanie po promieniu i statystyki organizacji to pojedyncze zapytania SQL z indeksami.
- Limit miejsc pilnuje blokada wiersza akcji w transakcji zapisu i awansu z listy rezerwowej.

### Negatywne
- Testy integracyjne muszą działać na prawdziwej bazie z PostGIS (kontener usługi w CI), a nie na SQLite, co wydłuża CI.

### Neutralne (zadania)
- Czasy zapisujemy jako `timestamptz` (UTC) i wyświetlamy w strefie Europe/Warsaw.
- Kopie zapasowe: cykliczny `pg_dump` poza serwer i opis odtworzenia w dokumentacji technicznej ([ADR-009](009-wdrozenie-docker-compose-i-ci.md)).
