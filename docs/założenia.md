[ZALOZENIA.md](https://github.com/user-attachments/files/34924580/ZALOZENIA.md)
# Założenia projektu: platforma do zarządzania wolopntariatem i akcjami społecznymi

**Temat projektu:** System wspierający organizację i obsługę wolontariatu: łączenie wolontariuszy z organizacjami, zarządzanie akcjami, rekomendacje i potwierdzenia udziału.

## 1. Role w systemie

| Rola | Opis |
|---|---|
| Gość | Osoba niezalogowana. Może przeglądać publiczne akcje i się zarejestrować. |
| Wolontariusz (użytkownik) | Zarejestrowany użytkownik, który zapisuje się na akcje i buduje historię aktywności. |
| Organizator (organizacja) | Zatwierdzona organizacja, która tworzy akcje i zarządza zgłoszeniami. |
| Administrator | Zarządza użytkownikami i organizacjami, zatwierdza organizacje, przegląda logi. |

## 2. Założenia funkcjonalne

### 2.1 Konta i uwierzytelnianie
-  Gość może założyć konto wolontariusza przez e-mail i hasło, z weryfikacją adresu e-mail linkiem aktywacyjnym.
-  Gość może zarejestrować się i zalogować przez zewnętrznego dostawcę OAuth 2.0 / OpenID Connect (np. Google).
-  Użytkownik może zresetować hasło przez e-mail.
-  Użytkownik może edytować dane konta i usunąć konto .

### 2.2 Profil wolontariusza
-  Użytkownik może wskazać w profilu posiadane umiejętności, wybierając je ze słownika.
-  Użytkownik może wskazać interesujące go rodzaje wolontariatu, np. pomoc społeczna, ekologia, edukacja, zwierzęta, sport, kultura.
-  Użytkownik może określić miejscowość lub obszar zainteresowania, jako miasto albo punkt na mapie z promieniem w km.

### 2.3 Organizacje
-  Użytkownik może utworzyć profil organizacji: nazwa, opis, logo, dane kontaktowe, adres, NIP/KRS (opcjonalnie), strona WWW.
-  Nowa organizacja otrzymuje status *oczekująca*. Do czasu zatwierdzenia nie może publikować akcji.
-  Administrator może zatwierdzić lub odrzucić organizację (z podaniem uzasadnienia) i zablokować ją później. Organizacja dostaje powiadomienie o decyzji.
-  Organizacja może mieć wielu członków zespołu z uprawnieniami organizatora (opcjonalnie).

### 2.4 Akcje wolontariackie
-  Zatwierdzona organizacja może tworzyć, edytować, anulować i usuwać akcje z polami:
  - nazwa,
  - opis,
  - miejsce ,
  - termin (data i godzina rozpoczęcia oraz zakończenia),
  - limit uczestników,
  - wymagane umiejętności .
-  Akcja ma statusy: *szkic, opublikowana, zakończona, anulowana*.
-  Przy zmianie terminu lub miejsca, albo przy anulowaniu akcji, system powiadamia zapisanych uczestników.

### 2.5 Wyszukiwanie
-  Użytkownik może wyszukiwać akcje po filtrach: słowa kluczowe, rodzaj wolontariatu, termin (od–do), miejscowość lub odległość od wskazanego punktu, wymagane umiejętności, dostępność miejsc, organizacja.
-  Wyniki są prezentowane jako lista i jako znaczniki na mapie. Filtry i widok mapy są zsynchronizowane.
-  Użytkownik może skorzystać z geolokalizacji przeglądarki, aby wyszukać akcje w pobliżu (za zgodą użytkownika).

### 2.6 Zapisy, lista rezerwowa i zgłoszenia
-  Zalogowany użytkownik może zapisać się na akcję i wycofać swoje zgłoszenie.
-  Gdy limit uczestników jest wyczerpany, zgłoszenie trafia na listę rezerwową z zachowaniem kolejności.
-  Gdy zwolni się miejsce (wycofanie lub odrzucenie zgłoszenia), pierwszy użytkownik z listy rezerwowej jest automatycznie awansowany, a system go o tym powiadamia.
-  Organizator może zatwierdzić lub odrzucić zgłoszenie (z opcjonalnym komentarzem). Użytkownik dostaje powiadomienie. Limit uczestników liczy się względem zgłoszeń zatwierdzonych.
-  System zapobiega zapisom na kolidujące czasowo akcje i podwójnym zapisom (ostrzeżenie lub blokada).

### 2.7 Realizacja i potwierdzenia udziału
-  Po zakończeniu akcji organizator oznacza dla każdego zatwierdzonego uczestnika faktyczną obecność (obecny / nieobecny) oraz może skorygować liczbę przepracowanych godzin (domyślnie czas trwania akcji).
-  Użytkownik widzi historię swoich aktywności (akcje, organizacje, daty, status) oraz sumę przepracowanych godzin, z filtrowaniem po okresie.
-  Organizator może wystawić elektroniczne potwierdzenie udziału. System generuje je jako plik PDF zawierający dane wolontariusza, organizacji i akcji, termin, liczbę godzin, datę wystawienia, unikalny numer oraz kod QR lub link weryfikacyjny.
-  Użytkownik może pobrać swoje potwierdzenia z poziomu profilu. Każde potwierdzenie można zweryfikować publicznie po numerze lub kodzie.

### 2.8 Obserwowanie i powiadomienia
-  Użytkownik może obserwować wybrane organizacje i przestać je obserwować.
-  System powiadamia o nowych akcjach obserwowanych organizacji, w aplikacji i e-mailem. Użytkownik może konfigurować kanały i częstotliwość powiadomień.
-  System wysyła powiadomienia transakcyjne: decyzja o zgłoszeniu, awans z listy rezerwowej, zmiana lub anulowanie akcji, przypomnienie przed akcją, wystawione potwierdzenie.

### 2.9 Rekomendacje
-  System rekomenduje użytkownikowi akcje na podstawie:
  - zainteresowań (rodzaje wolontariatu),
  - lokalizacji i obszaru zainteresowania,
  - dopasowania umiejętności do wymagań akcji,
  - wcześniejszej aktywności (np. organizacje i kategorie, w których już uczestniczył).
-  Rekomendacje są wyświetlane na stronie głównej użytkownika. Wskazuje się powód rekomendacji (np. „blisko Ciebie", „pasuje do Twoich umiejętności").

### 2.10 Statystyki organizacji
-  Organizacja widzi panel statystyk:
  - liczba unikalnych wolontariuszy,
  - frekwencja na wydarzeniach (obecni / zatwierdzeni, per akcja i łącznie),
  - liczba przepracowanych godzin.
-  Statystyki można filtrować po okresie i akcji. 

### 2.11 Administracja
-  Administrator może przeglądać, wyszukiwać, blokować i odblokowywać użytkowników oraz organizacje oraz zmieniać role.
-  Administrator ma kolejkę organizacji oczekujących na weryfikację.
-  Administrator ma wgląd w logi aplikacji z filtrowaniem po poziomie, czasie, użytkowniku i typie zdarzenia. Obejmują one zdarzenia biznesowe (rejestracja, zatwierdzenie organizacji, wystawienie potwierdzenia) i błędy techniczne.
-  Administrator zarządza słownikami (umiejętności, rodzaje wolontariatu).

## 3. Założenia niefunkcjonalne

### 3.1 Architektura i integracja
-  Backend i frontend są niezależnymi aplikacjami, wdrażanymi osobno.
-  Komunikacja odbywa się wyłącznie przez REST API (JSON over HTTPS), bezstanowe, z wersjonowaniem (`/api/v1`).
-  Backend jest udokumentowany w standardzie **OpenAPI 3.x**. Specyfikacja jest generowana lub utrzymywana wraz z kodem, a interaktywna dokumentacja (Swagger UI / Redoc) jest dostępna pod adresem `/docs`.
-  Frontend to aplikacja SPA, responsywna (desktop, tablet, telefon).

### 3.2 Mapa i geolokalizacja
-  System korzysta z dostawcy map (np. OpenStreetMap z Leaflet lub MapLibre, alternatywnie Google Maps) oraz z usługi geokodowania adresów.
-  Współrzędne akcji są przechowywane w bazie ze wsparciem zapytań przestrzennych (np. PostgreSQL + PostGIS) dla wyszukiwania po promieniu.
-  Geolokalizacja przeglądarki jest używana wyłącznie za zgodą użytkownika, a przy jej odmowie działa wyszukiwanie ręczne.

### 3.3 Bezpieczeństwo
-  Uwierzytelnianie oparte na tokenach (JWT z krótkim czasem życia i refresh tokenem lub sesje serwerowe) oraz OAuth 2.0 / OIDC dla logowania zewnętrznego.
-  Hasła są haszowane algorytmem adaptacyjnym (bcrypt lub Argon2). Wymagana jest minimalna siła hasła.
-  Autoryzacja oparta na rolach (RBAC) i kontrola własności zasobów (organizacja edytuje tylko własne akcje).
-  Dostęp do plików PDF potwierdzeń jest ograniczony do właściciela, organizacji wystawiającej i administratora. Weryfikacja publiczna ujawnia tylko dane minimalne.

### 3.4 Ochrona danych (RODO)
-  Zbierane są tylko dane niezbędne. System zapewnia zgody (regulamin, polityka prywatności, zgoda na lokalizację i powiadomienia), prawo do wglądu, eksportu i usunięcia danych.
-  Logi nie zawierają haseł, tokenów ani nadmiarowych danych osobowych.

### 3.5 Wydajność i skalowalność
-  Czas odpowiedzi API dla typowych operacji ≤ 500 ms (p95) przy 100 równoległych użytkownikach. Wyszukiwanie z mapą ≤ 1 s.
-  Listy są paginowane. Zapytania są wspierane indeksami (w tym przestrzennymi).
-  Generowanie PDF i wysyłka powiadomień odbywają się asynchronicznie (kolejka zadań) tam, gdzie to zasadne.

### 3.6 Niezawodność i spójność danych
-  Operacje zapisu na akcję i awansu z listy rezerwowej są transakcyjne i odporne na wyścigi (blokady lub kontrola współbieżności), aby nie przekroczyć limitu miejsc.
-  Kopie zapasowe bazy danych wykonywane cyklicznie, z udokumentowaną procedurą odtworzenia. Docelowa dostępność ≥ 99% (w warunkach projektu).

### 3.7 Logowanie i monitoring
-  Logi aplikacji są ustrukturyzowane (JSON), z poziomami, identyfikatorem żądania i znacznikiem czasu. Są dostępne w panelu administratora i przechowywane przez określony czas.
-  Endpoint health-check (`/health`) oraz podstawowe metryki działania.

### 3.8 Dokumenty PDF
-  Potwierdzenia są generowane serwerowo jako PDF (A4) z obsługą polskich znaków , zawierają unikalny numer  oraz są niezmienne po wystawieniu (ewentualna korekta tworzy nową wersję i unieważnia poprzednią).

### 3.9 Użyteczność i dostępność
-  Interfejs w języku polskim.
-  Obsługa aktualnych wersji Chrome, Firefox, Safari.

### 3.10 Jakość i utrzymanie
-  Testy jednostkowe i integracyjne backendu , testy kluczowych scenariuszy E2E.
-  Kod w repozytorium Git, CI/CD z automatycznym uruchamianiem testów i lintera, konfiguracja przez zmienne środowiskowe.
-  Wdrożenie w kontenerach (Docker, docker-compose), osobno frontend, backend i baza danych.

## 4. Przykładowy stos technoologiczny

| Warstwa | Przykład |
|---|---|
| Frontend | React / Vue / Angular + MapLibre lub Leaflet |
| Backend | Spring Boot (springdoc-openapi), ASP.NET Core, FastAPI lub NestJS |
| Baza danych | PostgreSQL + PostGIS |
| Uwierzytelnianie | OAuth 2.0 / OIDC (Google) + JWT |
| PDF | OpenPDF / iText, QuestPDF, WeasyPrint lub pdfkit (zależnie od stosu) |
| Kolejki i e-mail | Redis + worker, SMTP |
| Wdrożenie | Docker, GitHub Actions |


