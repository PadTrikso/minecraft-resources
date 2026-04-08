# PRD – Minecraft Monopoly Plugin

> **Wersja:** 1.0  
> **Data:** 2026-04-08  
> **Status:** Draft  

---

## 1. Executive Summary

### Problem Statement

Gracze Minecraft poszukują angażujących trybów rozrywki wieloosobowej bezpośrednio w grze, bez konieczności korzystania z zewnętrznych platform. Klasyczna gra Monopoly cieszy się ogromną popularnością, ale nie istnieje dojrzały plugin, który wiernie odwzorowuje jej mechaniki w środowisku Minecraft – z wykorzystaniem trójwymiarowej planszy, interaktywnych bloków i systemu ekonomicznego.

### Proposed Solution

Plugin **Minecraft Monopoly** przenosi pełne doświadczenie gry Monopoly do świata Minecraft. Gracze poruszają się po fizycznej, trójwymiarowej planszy zbudowanej z bloków, kupują nieruchomości, budują domy i hotele, handlują między sobą, płacą podatki i czynsze – wszystko zarządzane przez zautomatyzowany system bankowy zintegrowany z ekonomią serwera. Rozgrywka odbywa się w pełni w grze za pomocą komend, GUI (inventory menu) oraz interaktywnych elementów na planszy.

### Success Criteria (KPI)

| KPI | Cel | Metoda pomiaru |
|-----|-----|----------------|
| Stabilność rozgrywki | 0 krytycznych błędów (crash) na 100 rozegranych gier | Logi serwera, raporty błędów |
| Czas odpowiedzi systemu | Każda akcja gracza (rzut kostką, zakup, płatność) przetworzona w ≤ 200 ms | Profiler serwera |
| Średni czas rozgrywki | 20–60 minut na pełną grę (2–6 graczy) | Statystyki z zakończonych sesji |
| Adopcja przez graczy | ≥ 80% graczy, którzy rozpoczęli grę, kończy ją (nie opuszcza przed zakończeniem) | Telemetria sesji |
| Satysfakcja graczy | Ocena ≥ 4.0/5.0 w systemie feedbacku po grze | Ankieta in-game |

---

## 2. User Experience & Functionality

### 2.1 User Personas

| Persona | Opis | Potrzeby |
|---------|------|----------|
| **Gracz casualowy** | Gracz Minecraft w wieku 12–25 lat, gra z przyjaciółmi na prywatnym serwerze, zna podstawowe zasady Monopoly | Prosta obsługa, czytelne GUI, szybki start gry |
| **Administrator serwera** | Zarządza serwerem multiplayer (10–100 graczy), instaluje pluginy, konfiguruje rozgrywkę | Łatwa instalacja, konfigurowalne parametry gry, niskie zużycie zasobów |
| **Gracz kompetytywny** | Doświadczony gracz Monopoly, oczekuje wiernego odwzorowania zasad, chce strategii i rywalizacji | Pełna implementacja zasad, system rankingowy, statystyki |

### 2.2 User Stories & Acceptance Criteria

#### US-01: Rozpoczęcie gry
**Jako** gracz, **chcę** szybko rozpocząć sesję Monopoly z innymi graczami, **aby** nie tracić czasu na konfigurację.

**Acceptance Criteria:**
- Gracz tworzy lobby komendą `/monopoly create` i otrzymuje potwierdzenie w chacie
- Pozostali gracze dołączają komendą `/monopoly join <id>`
- Gra obsługuje od 2 do 6 graczy jednocześnie
- Host (twórca lobby) uruchamia grę komendą `/monopoly start`
- Każdy gracz wybiera pionek z dostępnej puli (za pomocą GUI inventory menu)
- Po starcie każdy gracz otrzymuje 1500 zł w wirtualnym portfelu (rozkład nominałów: 2×500, 4×100, 1×50, 1×20, 2×10, 1×5, 5×1)
- Wszyscy gracze zostają teleportowani na pole „Start" na trójwymiarowej planszy

#### US-02: Rzut kostką i poruszanie się
**Jako** gracz, **chcę** rzucać kostkami i poruszać się po planszy, **aby** odwiedzać kolejne pola.

**Acceptance Criteria:**
- Gracz rzuca kostkami komendą `/monopoly roll` lub kliknięciem dedykowanego przedmiotu w hotbarze
- System symuluje rzut dwiema sześciennymi kostkami (animacja cząsteczek nad planszą)
- Pionek gracza (armor stand lub entity) przesuwa się automatycznie o wylosowaną liczbę pól zgodnie z ruchem wskazówek zegara
- W przypadku wyrzucenia dubletu gracz otrzymuje dodatkowy rzut
- Trzy dublety pod rząd = automatyczne przeniesienie do więzienia (bez przechodzenia przez Start)
- Każdy gracz ma 50 sekund na wykonanie ruchu; po przekroczeniu limitu traci kolejkę
- Timer jest widoczny na ekranie gracza (action bar lub bossbar)

#### US-03: Zakup nieruchomości
**Jako** gracz, **chcę** kupować nieruchomości, na których staję, **aby** budować swój majątek.

**Acceptance Criteria:**
- Gdy gracz stanie na niewykupionym polu nieruchomości, otrzymuje GUI z opcjami: „Kup" / „Wystaw na aukcję"
- Kliknięcie „Kup" pobiera kwotę z portfela gracza i przypisuje mu kartę nieruchomości (widoczną w `/monopoly properties`)
- Kliknięcie „Wystaw na aukcję" uruchamia system licytacji:
  - Wszyscy gracze mogą składać oferty przez 30 sekund
  - Oferty składane są komendą `/monopoly bid <kwota>` lub przez GUI
  - Najwyższa oferta wygrywa; kwota jest pobierana z portfela zwycięzcy
  - Jeśli nikt nie złoży oferty, nieruchomość wraca do banku
- Gracz może sprzedać nieruchomość innemu graczowi komendą `/monopoly sell <nieruchomość> <gracz> <cena>` (wymaga akceptacji kupującego)

#### US-04: Pobieranie czynszu
**Jako** właściciel nieruchomości, **chcę** automatycznie otrzymywać czynsz od graczy stających na moich polach, **aby** pomnażać swój kapitał.

**Acceptance Criteria:**
- Gdy gracz stanie na polu posiadanym przez innego gracza, system automatycznie powiadamia właściciela
- Właściciel musi zatwierdzić pobranie czynszu komendą `/monopoly collect` lub kliknięciem w GUI (w ciągu 15 sekund)
- Jeśli właściciel nie zareaguje w ciągu 15 sekund, czynsz nie jest pobierany (symulacja „zapomnienia" – zgodnie z oryginalnymi zasadami)
- Kwota czynszu jest obliczana automatycznie na podstawie:
  - Bazowej stawki czynszu z karty nieruchomości
  - Liczby domów/hoteli na danym polu
  - Posiadania pełnego monopolu na kolor (podwójny czynsz bez budynków)
- Jeśli gracz nie ma wystarczających środków na czynsz → uruchomienie procedury bankructwa (US-09)

#### US-05: Budowanie domów i hoteli
**Jako** właściciel pełnej dzielnicy (wszystkie ulice jednego koloru), **chcę** budować domy i hotele, **aby** zwiększać czynsz.

**Acceptance Criteria:**
- Gracz może budować domy tylko jeśli posiada wszystkie nieruchomości danego koloru (monopol)
- Budowa odbywa się komendą `/monopoly build <nieruchomość>` lub przez GUI
- Domy muszą być budowane równomiernie: różnica w liczbie domów między ulicami tego samego koloru nie może przekraczać 1
- Maksymalnie 4 domy na jednej nieruchomości
- Po postawieniu 4 domów gracz może zbudować hotel (dopłata = cena 1 domu, zamiana 4 domów na 1 hotel)
- Maksymalnie 1 hotel na nieruchomość (hotel = najwyższy poziom zabudowy)
- Domy i hotele są fizycznie widoczne na planszy jako bloki (np. domy = bloki terakoty, hotel = blok złota)
- Gracz może budować w dowolnym momencie swojej tury, nie tylko stojąc na danym polu
- Bank ma ograniczoną liczbę domów (32) i hoteli (12) – jak w oryginalnej grze

#### US-06: Więzienie
**Jako** gracz, **chcę**, aby mechanika więzienia działała zgodnie z zasadami Monopoly, **aby** gra była strategicznie interesująca.

**Acceptance Criteria:**
- Gracz trafia do więzienia gdy:
  - Stanie na polu „Idź do więzienia"
  - Wyrzuci dublet 3 razy z rzędu
  - Wylosuje kartę „Idź do więzienia" z puli Szansa/Kasa Społeczna
- Gracz w więzieniu nie przechodzi przez Start (nie otrzymuje 200 zł)
- Sposoby wyjścia z więzienia:
  - Zapłacenie kaucji 50 zł (komenda `/monopoly bail`)
  - Użycie karty „Wyjście z więzienia" (`/monopoly usecard jail`)
  - Wyrzucenie dubletu w jednej z 3 następnych tur
- Po 3 nieudanych próbach wyrzucenia dubletu gracz musi zapłacić 50 zł kaucji
- Jeśli gracza nie stać na kaucję → bankructwo (US-09)
- Pionek gracza w więzieniu jest wizualnie oddzielony (np. za kratami z żelaznych prętów na planszy)

#### US-07: Karty Szansa i Kasa Społeczna
**Jako** gracz, **chcę** losować karty zdarzeń, **aby** gra była nieprzewidywalna i emocjonująca.

**Acceptance Criteria:**
- Po stanięciu na polu „Szansa" lub „Kasa Społeczna" system losuje kartę z odpowiedniego stosu
- Treść karty wyświetlana jest graczowi w GUI (book & quill style) oraz na chacie
- Efekt karty jest wykonywany automatycznie (np. przeniesienie na pole, otrzymanie/zapłata pieniędzy)
- Karty „Wyjście z więzienia" mogą być zachowane przez gracza i użyte później
- Po wykorzystaniu wszystkich kart stos jest automatycznie tasowany
- Zestaw kart jest konfigurowalny w pliku `cards.yml` przez administratora serwera
- Domyślny zestaw zawiera minimum 16 kart Szansy i 16 kart Kasy Społecznej

#### US-08: Podatki i premia za Start
**Jako** gracz, **chcę**, aby system podatkowy i premie działały automatycznie, **aby** gra przebiegała płynnie.

**Acceptance Criteria:**
- Każdy gracz przechodzący (lub stający na) pole „Start" otrzymuje automatycznie 200 zł od banku
- Pole „Podatek dochodowy" pobiera automatycznie 200 zł od gracza
- Pole „Domiar podatkowy" pobiera automatycznie 100 zł od gracza
- Wszystkie transakcje podatkowe są wyświetlane graczowi jako wiadomość na chacie z animacją (dźwięk + cząsteczki)
- Jeśli gracz nie ma wystarczających środków na podatek → uruchomienie procedury bankructwa (US-09)

#### US-09: Bankructwo i zakończenie gry
**Jako** gracz, **chcę**, aby mechanika bankructwa była czytelna i sprawiedliwa, **aby** wynik gry był jednoznaczny.

**Acceptance Criteria:**
- Gracz jest uznawany za bankruta, gdy nie może pokryć zobowiązania (czynsz, podatek, kaucja) i nie posiada żadnych aktywów do sprzedaży/zastawu
- Przed ogłoszeniem bankructwa gracz ma możliwość:
  - Sprzedaży domów/hoteli bankowi za 50% wartości budowy
  - Zastawienia nieruchomości w banku za 50% wartości (komenda `/monopoly mortgage <nieruchomość>`)
  - Sprzedaży nieruchomości innemu graczowi
- Jeśli dług jest wobec innego gracza → nieruchomości bankruta przechodzą na wierzyciela
- Jeśli dług jest wobec banku → nieruchomości wracają do banku (mogą być wystawione na aukcję)
- Bankrut jest usuwany z gry (teleportowany z planszy, otrzymuje komunikat podsumowujący)
- Gdy zostaje tylko 1 gracz → jest ogłaszany zwycięzcą (tytuł na ekranie, fajerwerki, dźwięk zwycięstwa)
- Alternatywne zakończenie: komenda `/monopoly end` (tylko host) kończy grę i wygrywa gracz z najwyższym łącznym majątkiem (gotówka + wartość nieruchomości + domy/hotele + zastawione nieruchomości po 50% ceny)

#### US-10: Kredyt bankowy (zastaw hipoteczny)
**Jako** gracz, **chcę** zastawiać nieruchomości w banku, **aby** uzyskać gotówkę w trudnej sytuacji.

**Acceptance Criteria:**
- Gracz może zastawić nieruchomość komendą `/monopoly mortgage <nieruchomość>`
- Kwota pożyczki = 50% wartości nieruchomości (widnieje na karcie)
- Zastawiona nieruchomość nie generuje czynszu (gracze stający na niej nie płacą)
- Wykupienie zastawu: `/monopoly unmortgage <nieruchomość>` – koszt = 50% wartości + 10% odsetek
- Zastawione nieruchomości są wizualnie oznaczone na planszy (np. czerwony banner)
- Przed zastawieniem nieruchomości z monopolu gracz musi sprzedać wszystkie domy/hotele na ulicach tego koloru
- Żaden gracz nie może pożyczać pieniędzy innemu graczowi (blokada systemowa)

#### US-11: Zarządzanie grą (Administrator)
**Jako** administrator serwera, **chcę** konfigurować parametry gry, **aby** dostosować rozgrywkę do potrzeb serwera.

**Acceptance Criteria:**
- Plik konfiguracyjny `config.yml` umożliwia ustawienie:
  - Liczby graczy (min/max)
  - Kwoty startowej
  - Czasu na ruch (domyślnie 50 sekund)
  - Premii za przejście Startu
  - Stawek podatkowych
  - Limitu domów i hoteli
  - Włączenia/wyłączenia systemu aukcji
- Plansza jest generowana automatycznie komendą `/monopoly setup` w wybranej lokalizacji
- Administrator może przerwać grę komendą `/monopoly forceend`
- Plugin loguje wszystkie transakcje do pliku `logs/monopoly.log`
- Plugin nie koliduje z innymi popularnymi pluginami ekonomicznymi (Vault API)

### 2.3 Non-Goals (Czego NIE budujemy w wersji MVP)

- **Tryb single-player z AI** – brak botów jako przeciwników
- **Integracja z prawdziwą walutą serwera** – plugin używa własnej waluty sesyjnej, nie wpływa na ekonomię serwera
- **Edytor planszy in-game** – plansza generowana jest automatycznie; niestandardowe plansze wymagają edycji plików konfiguracyjnych
- **Tryb spectator** – brak dedykowanego trybu obserwatora (gracze mogą obserwować planszę, ale bez dedykowanego UI)
- **Aplikacja mobilna / webowa** – plugin działa wyłącznie w Minecraft Java Edition
- **Cross-server play** – gra odbywa się na jednym serwerze, brak wsparcia dla BungeeCord/Velocity w MVP

---

## 3. Technical Specifications

### 3.1 Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Minecraft Server                      │
│                   (Spigot / Paper)                       │
│                                                         │
│  ┌───────────────────────────────────────────────────┐  │
│  │              Minecraft Monopoly Plugin             │  │
│  │                                                   │  │
│  │  ┌─────────────┐  ┌──────────────┐  ┌──────────┐ │  │
│  │  │  Game       │  │  Board       │  │  Bank    │ │  │
│  │  │  Manager    │  │  Manager     │  │  System  │ │  │
│  │  │             │  │              │  │          │ │  │
│  │  │ - Lobby     │  │ - Generation │  │ - Wallet │ │  │
│  │  │ - Turns     │  │ - Fields     │  │ - Taxes  │ │  │
│  │  │ - Players   │  │ - Movement   │  │ - Loans  │ │  │
│  │  │ - Rules     │  │ - Visuals    │  │ - Audit  │ │  │
│  │  └──────┬──────┘  └──────┬───────┘  └────┬─────┘ │  │
│  │         │                │               │        │  │
│  │  ┌──────┴────────────────┴───────────────┴─────┐  │  │
│  │  │              Event Bus / Listener            │  │  │
│  │  └──────────────────┬──────────────────────────┘  │  │
│  │                     │                             │  │
│  │  ┌─────────────┐  ┌┴─────────────┐  ┌──────────┐ │  │
│  │  │  Card       │  │  Property    │  │  GUI     │ │  │
│  │  │  System     │  │  Manager     │  │  Manager │ │  │
│  │  │             │  │              │  │          │ │  │
│  │  │ - Chance    │  │ - Ownership  │  │ - Menus  │ │  │
│  │  │ - Community │  │ - Buildings  │  │ - Cards  │ │  │
│  │  │ - Effects   │  │ - Mortgage   │  │ - Info   │ │  │
│  │  └─────────────┘  └──────────────┘  └──────────┘ │  │
│  │                                                   │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │           Configuration & Storage            │  │  │
│  │  │  config.yml │ cards.yml │ board.yml │ SQLite │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────┘  │
│                                                         │
│  ┌────────────────┐  ┌────────────────────────────────┐ │
│  │  Vault API     │  │  Spigot/Paper API              │ │
│  │  (opcjonalne)  │  │  (events, commands, scheduler) │ │
│  └────────────────┘  └────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

### 3.2 Główne komponenty

| Komponent | Odpowiedzialność |
|-----------|-----------------|
| **GameManager** | Zarządzanie sesjami gry (lobby, start, zakończenie), kolejnością tur, stanem gry |
| **BoardManager** | Generowanie planszy 3D z bloków, mapowanie pól, zarządzanie wizualizacją pionków |
| **BankSystem** | Portfele graczy, transakcje, podatki, pożyczki hipoteczne, audyt |
| **PropertyManager** | Własność nieruchomości, budowa domów/hoteli, licytacje, handel między graczami |
| **CardSystem** | Losowanie i wykonywanie kart Szansa/Kasa Społeczna, konfiguracja zestawów |
| **GUIManager** | Inventory-based menu (zakup, aukcja, budowa, informacje o nieruchomościach) |
| **Configuration & Storage** | Pliki YAML (config, karty, plansza), SQLite dla statystyk i logów |

### 3.3 Technology Stack

| Element | Technologia |
|---------|-------------|
| Język programowania | Java 17+ |
| Framework serwera | Spigot API / Paper API (1.20.x – 1.21.x) |
| System budowania | Maven lub Gradle |
| Baza danych (statystyki) | SQLite (wbudowana, bez zewnętrznych zależności) |
| Konfiguracja | YAML (SnakeYAML – wbudowane w Spigot) |
| GUI | Bukkit Inventory API (Chest GUI) |
| Ekonomia (opcjonalnie) | Vault API (soft dependency) |
| Wersjonowanie | Git + Semantic Versioning |

### 3.4 Integration Points

| Integracja | Typ | Opis |
|------------|-----|------|
| **Spigot/Paper API** | Core | Eventy (PlayerMoveEvent, InventoryClickEvent), komendy, scheduler |
| **Vault API** | Soft Dependency | Opcjonalna integracja z ekonomią serwera (nagrody za wygranie gry) |
| **PlaceholderAPI** | Soft Dependency | Placeholdery do scoreboard/tablist (np. `%monopoly_balance%`, `%monopoly_properties%`) |
| **SQLite** | Internal | Przechowywanie statystyk graczy (wygrane, przegrane, łączny zarobek) |
| **bStats** | Metrics | Anonimowe statystyki użycia pluginu |

### 3.5 Struktura plików konfiguracyjnych

#### config.yml
```yaml
game:
  min-players: 2
  max-players: 6
  starting-money: 1500
  turn-timeout-seconds: 50
  pass-go-bonus: 200
  jail-bail-cost: 50
  max-houses: 32
  max-hotels: 12
  auction-enabled: true
  auction-duration-seconds: 30
  collect-rent-timeout-seconds: 15

board:
  world: "world"
  origin:
    x: 0
    y: 64
    z: 0
  field-size: 5  # blocks per field

taxes:
  income-tax: 200
  luxury-tax: 100

messages:
  language: "pl"
```

#### cards.yml (przykład)
```yaml
chance:
  - id: "chance_01"
    text: "Bank wypłaca Ci dywidendę w wysokości 50 zł."
    action: "give_money"
    amount: 50
  - id: "chance_02"
    text: "Idź do więzienia. Nie przechodź przez Start."
    action: "go_to_jail"
  - id: "chance_03"
    text: "Przejdź na pole Start. Pobierz 200 zł."
    action: "move_to"
    target: "start"

community_chest:
  - id: "cc_01"
    text: "Błąd bankowy na Twoją korzyść. Pobierz 200 zł."
    action: "give_money"
    amount: 200
  - id: "cc_02"
    text: "Zapłać szpitalny rachunek – 100 zł."
    action: "take_money"
    amount: 100
  - id: "cc_03"
    text: "Wychodzisz z więzienia za darmo."
    action: "get_out_of_jail_card"
```

### 3.6 Komendy pluginu

| Komenda | Uprawnienie | Opis |
|---------|-------------|------|
| `/monopoly create` | `monopoly.play` | Tworzenie nowego lobby |
| `/monopoly join <id>` | `monopoly.play` | Dołączenie do lobby |
| `/monopoly start` | `monopoly.play` | Rozpoczęcie gry (tylko host) |
| `/monopoly roll` | `monopoly.play` | Rzut kostkami |
| `/monopoly buy` | `monopoly.play` | Zakup nieruchomości |
| `/monopoly auction` | `monopoly.play` | Wystawienie nieruchomości na aukcję |
| `/monopoly bid <kwota>` | `monopoly.play` | Złożenie oferty na aukcji |
| `/monopoly build <nieruchomość>` | `monopoly.play` | Budowa domu na nieruchomości |
| `/monopoly sell <nieruchomość> <gracz> <cena>` | `monopoly.play` | Sprzedaż nieruchomości graczowi |
| `/monopoly mortgage <nieruchomość>` | `monopoly.play` | Zastaw nieruchomości w banku |
| `/monopoly unmortgage <nieruchomość>` | `monopoly.play` | Wykup zastawionej nieruchomości |
| `/monopoly bail` | `monopoly.play` | Zapłacenie kaucji za wyjście z więzienia |
| `/monopoly usecard jail` | `monopoly.play` | Użycie karty wyjścia z więzienia |
| `/monopoly collect` | `monopoly.play` | Pobranie czynszu od gracza na swoim polu |
| `/monopoly properties` | `monopoly.play` | Podgląd posiadanych nieruchomości |
| `/monopoly balance` | `monopoly.play` | Sprawdzenie stanu portfela |
| `/monopoly end` | `monopoly.play` | Zakończenie gry (tylko host) |
| `/monopoly stats [gracz]` | `monopoly.play` | Statystyki gracza |
| `/monopoly setup` | `monopoly.admin` | Generowanie planszy |
| `/monopoly forceend` | `monopoly.admin` | Wymuszenie zakończenia gry |
| `/monopoly reload` | `monopoly.admin` | Przeładowanie konfiguracji |

### 3.7 Security & Privacy

- **Brak przechowywania danych osobowych** – plugin przechowuje jedynie UUID graczy Minecraft i statystyki gry
- **Walidacja danych wejściowych** – wszystkie argumenty komend są walidowane (sanityzacja stringów, kontrola zakresów numerycznych)
- **Zapobieganie exploitom** – system blokuje:
  - Duplikację pieniędzy (atomowe transakcje)
  - Podwójne kliknięcia w GUI (cooldown 200 ms)
  - Manipulację kolejnością tur (server-side state machine)
  - Nieautoryzowane komendy (sprawdzanie stanu gry przed wykonaniem akcji)
- **Rate limiting** – maksymalnie 10 komend na sekundę na gracza
- **Izolacja sesji** – każda gra Monopoly jest niezależną sesją; gracze nie mogą wpływać na inne sesje
- **Bezpieczne przechowywanie danych** – SQLite z parametryzowanymi zapytaniami (ochrona przed SQL injection)

---

## 4. Definicja planszy

### 4.1 Układ pól (40 pól, zgodnie z klasycznym Monopoly)

| # | Nazwa pola | Typ | Kolor | Cena zakupu | Czynsz bazowy |
|---|-----------|-----|-------|-------------|---------------|
| 0 | START | Specjalne | – | – | +200 zł (premia) |
| 1 | Ulica Konopacka | Nieruchomość | Brązowy | 60 zł | 2 zł |
| 2 | Kasa Społeczna | Karta | – | – | – |
| 3 | Ulica Stalowa | Nieruchomość | Brązowy | 60 zł | 4 zł |
| 4 | Podatek dochodowy | Podatek | – | – | 200 zł |
| 5 | Dworzec Zachodni | Dworzec | – | 200 zł | 25 zł |
| 6 | Ulica Radzymińska | Nieruchomość | Jasnoniebieski | 100 zł | 6 zł |
| 7 | Szansa | Karta | – | – | – |
| 8 | Ulica Jagiellońska | Nieruchomość | Jasnoniebieski | 100 zł | 6 zł |
| 9 | Ulica Targowa | Nieruchomość | Jasnoniebieski | 120 zł | 8 zł |
| 10 | Więzienie / Odwiedziny | Specjalne | – | – | – |
| 11 | Ulica Płowiecka | Nieruchomość | Różowy | 140 zł | 10 zł |
| 12 | Elektrownia | Użyteczność | – | 150 zł | 4× kostka |
| 13 | Ulica Marszałkowska | Nieruchomość | Różowy | 140 zł | 10 zł |
| 14 | Ulica Szeroka | Nieruchomość | Różowy | 160 zł | 12 zł |
| 15 | Dworzec Gdański | Dworzec | – | 200 zł | 25 zł |
| 16 | Ulica Mickiewicza | Nieruchomość | Pomarańczowy | 180 zł | 14 zł |
| 17 | Kasa Społeczna | Karta | – | – | – |
| 18 | Ulica Słowackiego | Nieruchomość | Pomarańczowy | 180 zł | 14 zł |
| 19 | Plac Wilsona | Nieruchomość | Pomarańczowy | 200 zł | 16 zł |
| 20 | Parking | Specjalne | – | – | – |
| 21 | Ulica Świętokrzyska | Nieruchomość | Czerwony | 220 zł | 18 zł |
| 22 | Szansa | Karta | – | – | – |
| 23 | Krakowskie Przedmieście | Nieruchomość | Czerwony | 220 zł | 18 zł |
| 24 | Nowy Świat | Nieruchomość | Czerwony | 240 zł | 20 zł |
| 25 | Dworzec Wschodni | Dworzec | – | 200 zł | 25 zł |
| 26 | Ulica Puławska | Nieruchomość | Żółty | 260 zł | 22 zł |
| 27 | Ulica Marszałkowska | Nieruchomość | Żółty | 260 zł | 22 zł |
| 28 | Wodociągi | Użyteczność | – | 150 zł | 4× kostka |
| 29 | Ulica Belwederska | Nieruchomość | Żółty | 280 zł | 24 zł |
| 30 | Idź do więzienia | Specjalne | – | – | – |
| 31 | Ulica Senatorska | Nieruchomość | Zielony | 300 zł | 26 zł |
| 32 | Ulica Miodowa | Nieruchomość | Zielony | 300 zł | 26 zł |
| 33 | Kasa Społeczna | Karta | – | – | – |
| 34 | Plac Trzech Krzyży | Nieruchomość | Zielony | 320 zł | 28 zł |
| 35 | Dworzec Centralny | Dworzec | – | 200 zł | 25 zł |
| 36 | Szansa | Karta | – | – | – |
| 37 | Aleje Jerozolimskie | Nieruchomość | Granatowy | 350 zł | 35 zł |
| 38 | Domiar podatkowy | Podatek | – | – | 100 zł |
| 39 | Aleje Ujazdowskie | Nieruchomość | Granatowy | 400 zł | 50 zł |

### 4.2 Tabela czynszów (przykład – Ulica Konopacka, kolor Brązowy)

| Stan | Czynsz |
|------|--------|
| Pusta (brak monopolu) | 2 zł |
| Pusta (z monopolem na kolor) | 4 zł (podwójny) |
| 1 dom | 10 zł |
| 2 domy | 30 zł |
| 3 domy | 90 zł |
| 4 domy | 160 zł |
| Hotel | 250 zł |

### 4.3 Dworce – tabela czynszów

| Posiadane dworce | Czynsz |
|------------------|--------|
| 1 dworzec | 25 zł |
| 2 dworce | 50 zł |
| 3 dworce | 100 zł |
| 4 dworce | 200 zł |

### 4.4 Użyteczność publiczna – zasady czynszu

| Posiadane zakłady | Czynsz |
|-------------------|--------|
| 1 zakład | 4 × wartość rzutu kostkami |
| 2 zakłady | 10 × wartość rzutu kostkami |

---

## 5. Risks & Roadmap

### 5.1 Phased Rollout

#### Faza 1 – MVP (v1.0) – szacowany czas: 8–12 tygodni
- Generowanie planszy 3D
- System lobby i zarządzanie graczami (2–6 graczy)
- Rzut kostkami + poruszanie się po planszy
- Zakup nieruchomości (bez aukcji)
- Pobieranie czynszu (automatyczne)
- Podstawowy system bankowy (portfel, transakcje)
- Premia za Start, podatki
- Więzienie (pełna mechanika)
- Bankructwo i zakończenie gry
- Konfiguracja YAML (config.yml)
- Podstawowe GUI (inventory menu)

#### Faza 2 – v1.1 – szacowany czas: 4–6 tygodni
- System aukcji / licytacji
- Karty Szansa i Kasa Społeczna (konfigurowalny zestaw)
- Budowa domów i hoteli (z wizualizacją na planszy)
- Zastaw hipoteczny (mortgage)
- Handel między graczami
- PlaceholderAPI support
- System wielojęzyczny (messages.yml)

#### Faza 3 – v2.0 – szacowany czas: 6–8 tygodni
- System rankingowy i statystyki (SQLite)
- Tryb spectator
- Niestandardowe plansze (edytor board.yml)
- Vault API integration (nagrody)
- bStats metrics
- Tryb szybkiej gry (skrócone zasady, mniejsza plansza)
- Optymalizacja wydajności dla wielu jednoczesnych sesji

### 5.2 Technical Risks

| Ryzyko | Prawdopodobieństwo | Wpływ | Mitygacja |
|--------|-------------------|-------|-----------|
| **Wydajność planszy 3D** – generowanie i aktualizacja dużej ilości bloków może obciążać serwer | Średnie | Wysoki | Asynchroniczne operacje na blokach (scheduler), ograniczenie animacji do widocznych graczy, chunk loading control |
| **Desynchronizacja stanu gry** – gracz opuszcza serwer w trakcie gry | Wysokie | Średni | Auto-save stanu gry co turę, system reconnect (gracz ma 5 minut na powrót), AI/auto-skip po timeout |
| **Konflikty z innymi pluginami** – modyfikacja bloków planszy przez inne pluginy (WorldGuard, GriefPrevention) | Średnie | Średni | Region protection dla planszy, soft dependency na WorldGuard, dokumentacja kompatybilności |
| **Race conditions w transakcjach** – równoczesne operacje bankowe prowadzące do duplikacji | Niskie | Wysoki | Synchronizacja transakcji (synchronized blocks), atomowe operacje na portfelach, audit log |
| **Problemy z GUI (inventory)** – exploity związane z shift-click, drag & drop | Średnie | Średni | Cancellowanie wszystkich nieautoryzowanych InventoryClickEvent, cooldown na kliknięcia |
| **Kompatybilność z wersjami MC** – zmiany w API Spigot/Paper między wersjami | Wysokie | Średni | Abstrakcja NMS, testowanie na wielu wersjach (CI/CD), wsparcie tylko dla 2 ostatnich major versions |
| **Pamięć RAM** – wiele jednoczesnych sesji gry na dużym serwerze | Niskie | Średni | Limit jednoczesnych sesji (konfigurowalny), lazy loading, czyszczenie zakończonych sesji |

### 5.3 Definition of Done (ogólna)

- [ ] Wszystkie user stories z Fazy 1 zaimplementowane i przetestowane
- [ ] Testy jednostkowe pokrywające ≥ 70% logiki biznesowej
- [ ] Testy integracyjne na serwerze Paper (najnowsza stabilna wersja)
- [ ] Dokumentacja użytkownika (README.md z instrukcją instalacji i konfiguracji)
- [ ] Dokumentacja komend (wiki lub `/monopoly help`)
- [ ] Code review przez minimum 1 osobę
- [ ] Brak znanych krytycznych błędów (P0/P1)
- [ ] Pliki konfiguracyjne z komentarzami wyjaśniającymi każdą opcję
- [ ] Plugin przetestowany z 2–6 graczami w sesji trwającej ≥ 30 minut

---

## 6. Appendix

### 6.1 Słownik pojęć

| Termin | Definicja |
|--------|-----------|
| **Monopol** | Posiadanie wszystkich nieruchomości jednego koloru |
| **Dublet** | Wyrzucenie tej samej liczby oczek na obu kostkach |
| **Kaucja** | Opłata 50 zł za wyjście z więzienia |
| **Zastaw (mortgage)** | Oddanie nieruchomości bankowi za 50% wartości w zamian za gotówkę |
| **Czynsz** | Opłata, którą gracz płaci właścicielowi pola, na którym stanął |
| **Bankructwo** | Stan, w którym gracz nie może pokryć zobowiązań i odpada z gry |
| **Lobby** | Poczekalnia przed rozpoczęciem gry, w której gracze dołączają do sesji |
| **Host** | Gracz, który utworzył lobby i ma uprawnienia do zarządzania sesją |

### 6.2 Referencje

- Oficjalne zasady gry Monopoly (Hasbro)
- Spigot API Documentation: https://hub.spigotmc.org/javadocs/spigot/
- Paper API Documentation: https://jd.papermc.io/paper/
- Vault API: https://github.com/MilkBowl/VaultAPI
