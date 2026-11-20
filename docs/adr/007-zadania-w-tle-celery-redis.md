# ADR-007: Zadania w tle w Celery z Redisem, powiadomienia w aplikacji i e-mailem

- **Status:** Proponowana
- **Data:** 2026-11-20
- **Autorzy:** Maciej Bułhak (propozycja do dyskusji w zespole)

---

## 1. Kontekst

- Powiadomienia transakcyjne: decyzja o zgłoszeniu, awans z listy rezerwowej, zmiana lub anulowanie akcji, przypomnienie przed akcją, wystawione potwierdzenie (2.8).
- Powiadomienia o nowych akcjach obserwowanych organizacji, w aplikacji i e-mailem, z wyborem kanałów i częstotliwości (2.8).
- Generowanie PDF i wysyłka powiadomień mają działać asynchronicznie (3.5).
- Zadania cykliczne: przypomnienia przed akcją, zmiana statusu akcji na „zakończona” po terminie, podsumowania powiadomień.
- Backend w Django ([ADR-002](002-backend-django-rest-framework.md)), wdrożenie w Docker Compose ([ADR-009](009-wdrozenie-docker-compose-i-ci.md)).

---

## 2. Rozważane opcje

### Opcja 1: Celery z Redisem jako brokerem i Celery Beat do zadań cyklicznych

- **Zalety:**
  - [+] Standard w projektach Django, z dobrą dokumentacją i wieloma przykładami.
  - [+] Ponawianie nieudanych zadań, np. przy chwilowej awarii serwera SMTP.
  - [+] Celery Beat uruchamia zadania według harmonogramu (przypomnienia, zamykanie akcji).
  - [+] Taki wariant (Redis i worker) podają założenia w przykładowym stosie.
- **Wady:**
  - [-] Trzy dodatkowe usługi w docker-compose: `redis`, `worker` i `beat` (worker i beat korzystają z tego samego obrazu co backend).
  - [-] Zadanie wrzucone do kolejki przed zatwierdzeniem transakcji może nie zobaczyć danych; trzeba używać `transaction.on_commit`.

### Opcja 2: Django-Q2 z kolejką w bazie danych

- **Zalety:**
  - [+] Kolejka i harmonogram bez Redisa, w istniejącej bazie PostgreSQL.
- **Wady:**
  - [-] Mniejsza społeczność i mniej przykładów niż Celery.
  - [-] Kolejka obciąża bazę danych (przy naszym ruchu pomijalnie).

### Opcja 3: Wysyłka synchroniczna w żądaniu HTTP i komendy Django uruchamiane przez cron

- **Zalety:**
  - [+] Najprostsza implementacja.
- **Wady:**
  - [-] Wolne odpowiedzi API, a powiadomienia giną przy awarii SMTP, bo nie ma ponawiania.
  - [-] Niezgodne z wymaganiem asynchronicznej wysyłki (3.5).

---

## 3. Decyzja

**Wybieramy Celery z Redisem i Celery Beat. Powiadomienia w aplikacji zapisujemy w bazie, a frontend odpytuje o nieprzeczytane co 30–60 sekund. E-maile wysyłamy przez SMTP: lokalnie do Mailpit, na produkcji do dostawcy SMTP z darmowym limitem.**

---

## 4. Uzasadnienie

Celery jest najlepiej udokumentowanym rozwiązaniem dla Django i daje wszystko, czego potrzebujemy: kolejkę, ponawianie i harmonogram. Dodatkowe usługi w docker-compose kosztują niewiele, bo worker i beat to ten sam obraz co backend. Django-Q2 oszczędza Redisa, ale ma mniejszą społeczność, a przy problemach liczy się dostępność przykładów. Odpytywanie o powiadomienia co kilkadziesiąt sekund wystarcza do powiadomień o zapisach i akcjach; połączenie na żywo (Server-Sent Events) można dodać później bez zmiany modelu danych.

---

## 5. Konsekwencje

### Pozytywne
- Jedno miejsce na wszystkie zadania w tle: e-maile, PDF ([ADR-008](008-pdf-potwierdzen-weasyprint.md)), przypomnienia, zamykanie akcji.
- Mailpit pozwala testować e-maile lokalnie bez wysyłania ich naprawdę.

### Negatywne
- Powiadomienie w aplikacji pojawia się z opóźnieniem do minuty.
- Więcej kontenerów do uruchomienia lokalnie i do obserwowania na serwerze.

### Neutralne (zadania)
- W testach zadania wykonują się od razu, bez Redisa (`CELERY_TASK_ALWAYS_EAGER`).
- Wybór dostawcy SMTP na produkcję przy wdrożeniu ([ADR-009](009-wdrozenie-docker-compose-i-ci.md)); dane logowania tylko w zmiennych środowiskowych.
- Do ustalenia: z jakim wyprzedzeniem przypomnienie przed akcją (propozycja: 24 godziny) i jakie częstotliwości powiadomień o nowych akcjach (propozycja: od razu albo raz dziennie).
