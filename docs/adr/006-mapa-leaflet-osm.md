# ADR-006: Mapa w Leaflet z kafelkami OpenStreetMap, geokodowanie przez Nominatim

- **Status:** Proponowana
- **Data:** 2026-11-20
- **Autorzy:** Maciej Bułhak (propozycja do dyskusji w zespole)

---

## 1. Kontekst

- Wyniki wyszukiwania jako lista i znaczniki na mapie, z filtrami zsynchronizowanymi z widokiem mapy (2.5).
- Geolokalizacja przeglądarki tylko za zgodą; po odmowie działa wyszukiwanie ręczne (2.5, 3.2).
- Adres akcji i obszar wolontariusza (miasto albo punkt z promieniem) trzeba zamienić na współrzędne, czyli zgeokodować (2.2, 3.2).
- Projekt studencki bez budżetu, z małym ruchem.

---

## 2. Rozważane opcje

### Opcja 1: Leaflet (react-leaflet), kafelki OpenStreetMap, geokodowanie Nominatim

- **Zalety:**
  - [+] Darmowe, bez klucza API i bez karty płatniczej.
  - [+] Prosta, lekka biblioteka; grupowanie znaczników przez Leaflet.markercluster.
- **Wady:**
  - [-] Publiczne serwery OSM mają zasady użycia: widoczna atrybucja, identyfikacja aplikacji, mały ruch.
  - [-] Publiczny Nominatim: najwyżej 1 zapytanie na sekundę, zakaz podpowiedzi adresu przy wpisywaniu, wyniki trzeba zapisywać.
  - [-] Kafelki rastrowe słabo dają się dopasować kolorystycznie do design systemu.

### Opcja 2: MapLibre GL JS z kafelkami wektorowymi (np. MapTiler)

- **Zalety:**
  - [+] Mapę można ostylować kolorami z design systemu.
- **Wady:**
  - [-] Dostawca kafelków wektorowych wymaga klucza API i ma limity darmowego planu.
  - [-] Cięższa biblioteka (WebGL), wolniejsza na słabszych telefonach.

### Opcja 3: Google Maps Platform

- **Zalety:**
  - [+] Najlepsze geokodowanie i podpowiedzi adresów.
- **Wady:**
  - [-] Wymaga konta rozliczeniowego z kartą płatniczą; po przekroczeniu darmowego limitu płacimy.
  - [-] Uzależnienie od jednego dostawcy.

---

## 3. Decyzja

**Wybieramy Leaflet (react-leaflet) z kafelkami OpenStreetMap. Geokodowanie robi backend przez Nominatim, raz przy zapisie adresu, i zapisuje wynik w bazie.**

---

## 4. Uzasadnienie

Przy naszym ruchu darmowe usługi OSM wystarczają, a ich ograniczenia omijamy architekturą: frontend dostaje gotowe współrzędne z API, a backend geokoduje tylko przy zapisie akcji, organizacji albo profilu, czyli rzadko. MapLibre daje ładniejszą mapę kosztem klucza API i limitów, a Google Maps wymaga karty płatniczej.

---

## 5. Konsekwencje

### Pozytywne
- Mapa i geokodowanie bez kosztów i bez kluczy API.
- Wyszukiwanie po promieniu działa na współrzędnych w bazie ([ADR-004](004-baza-postgresql-postgis.md)), niezależnie od dostawcy mapy.

### Negatywne
- Brak podpowiedzi adresu przy wpisywaniu. Mitygacja: alternatywnie wskazanie punktu kliknięciem na mapie. Jeśli podpowiedzi okażą się potrzebne, trzeba będzie użyć innej usługi (np. Photon) albo własnej instancji Nominatim.

### Neutralne (zadania)
- Atrybucja „© OpenStreetMap contributors” widoczna na mapie (wymóg licencji ODbL).
- Backend wysyła do Nominatim nagłówek `User-Agent` z nazwą aplikacji i ogranicza się do 1 zapytania na sekundę.
