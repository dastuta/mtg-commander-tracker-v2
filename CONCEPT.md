# MTG Commander Tracker - Konzeptdokument

> **Version**: 0.1.0 (Draft)  
> **Status**: In Planung  
> **Zuletzt aktualisiert**: 2026-03-23

---

## Überblick

Dieses Dokument beschreibt die Konzeption und Architektur einer Open-Source Progressive Web App (PWA) zum Tracking von Magic: The Gathering Commander Spielen.

### Zielsetzung

1. **Einfacher Einstieg**: Casual-Spielern eine intuitive App bieten
2. **Erweiterbarkeit**: Architektur erlaubt schrittweise Feature-Erweiterungen
3. **Interoperabilität**: Export-Schnittstelle für Statistik-Tools
4. **Community-Driven**: Open Source, Beiträge willkommen

### Reifegrad-Modell

Die App folgt einem **Reifegrad-Modell** (Maturity Model). Die Kern-Architektur erlaubt von Anfang an die Erweiterung zu höheren Reifegraden, ohne das Grundkonzept zu brechen.

```
┌─────────────────────────────────────────────────────────────┐
│                      REIFEGRAD-MODELL                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   Grad 1: Prototyp    ──►  Grad 2: Produktiv                │
│   ├── Lebenstracker  │      ├── + Poison/CMD Damage         │
│   ├── JSON Export    │      ├── + Spieler-Datenbank         │
│   └── Offline-first  │      ├── + Zug-Timer                 │
│                      │      └── + Action Log                │
│                            │                                │
│                            ▼                                │
│                       Grad 3: Vernetzt                      │
│                       ├── + Multiplayer Sync                │
│                       ├── + Server-Backend                  │
│                       └── + Community-Features              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Reifegrade im Detail

### Grad 1: Prototyp (MVP)

**Ziel**: Minimal funktionsfähiges Produkt für erste Tests und Feedback.

#### Features

| Feature | Beschreibung | Priorität |
|---------|-------------|-----------|
| Lebenstracker | 4 Spieler, +/− Buttons | Must Have |
| JSON Export | Spielstand als JSON exportieren | Must Have |
| Offline-first | localStorage für Datenspeicherung | Must Have |
| PWA | Installierbar auf Mobile/Desktop | Should Have |
| Dark Mode | Standard dunkles Theme | Nice to Have |

#### Datenmodell (Grad 1)

```typescript
interface Game {
  id: string;
  players: Player[];
  finalState: FinalState;
  winner: string | null;
  duration: number; // Sekunden
}

interface Player {
  id: string;
  name: string;
  life: number;
}

interface FinalState {
  life: Record<string, number>; // playerId -> life
}
```

#### Tech Stack (Grad 1)

- **Framework**: Vue 3 (Composition API)
- **Build**: Vite
- **PWA**: vite-plugin-pwa
- **Styling**: Vanilla CSS (kein Framework)
- **Speicher**: localStorage

---

### Grad 2: Produktiv

**Ziel**: Vollständiger Commander-Tracker mit allen wichtigen Features.

#### Neue Features

| Feature | Beschreibung | Priorität |
|---------|-------------|-----------|
| Poison Counters | Gift-Zähler pro Spieler | Must Have |
| Commander Damage | Schaden pro Spieler-Paar | Must Have |
| Action Log | Chronologische Aktionshistorie | Must Have |
| Zug-Timer | Pro Zug und Gesamt | Should Have |
| Spieler-DB | Lokale Spieler-Verwaltung | Should Have |
| Undo/Redo | Aktionen rückgängig machen | Should Have |

#### Erweitertes Datenmodell (Grad 2)

```typescript
interface Game {
  id: string;
  version: string;
  createdAt: string; // ISO 8601
  
  players: Player[];
  actions: Action[];
  turns: Turn[];
  
  finalState: FinalState;
  winner: Winner | null;
  duration: number;
}

interface Player {
  id: string;
  name: string;
  commander: string | null;
  seat: number;
}

interface Action {
  id: string;
  turnNumber: number;
  timestamp: string;
  type: 'damage' | 'heal' | 'poison' | 'commander_damage' | 'defeat' | 'victory';
  source: { id: string; name: string } | null;
  target: { id: string; name: string };
  value: number;
  previousLife?: number;
  resultingLife?: number;
}

interface Turn {
  number: number;
  playerId: string;
  startTime: string;
  endTime: string | null;
}

interface FinalState {
  life: Record<string, number>;
  poison: Record<string, number>;
  commanderDamage: Record<string, number>; // "sourceId-targetId" -> damage
}

interface Winner {
  playerId: string;
  reason: 'last_standing' | 'concession' | 'other';
}
```

#### UI-Layout (Grad 2)

```
┌─────────────────────────────────────────────────────┐
│  [Timer]              GAME            [Menu ☰]     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │         Player 1 (Commander Name)            │   │
│  │  [Gift]    40 Leben    [CMD: 0/21]          │   │
│  │            ▲/▼  +5/-5                        │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │         Player 2 (Commander Name)            │   │
│  │  [Gift]    38 Leben    [CMD: 7/21]         │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │         Player 3 (Commander Name)            │   │
│  │  [Gift]    25 Leben    [CMD: 2/21]         │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │         Player 4 (Commander Name)            │   │
│  │  [Gift]    12 Leben    [CMD: 14/21]        │   │
│  └─────────────────────────────────────────────┘   │
│                                                     │
├─────────────────────────────────────────────────────┤
│  [◀ Undo]    Zug 7 (Spieler 2)    [Spiel beenden ▶] │
└─────────────────────────────────────────────────────┘
```

---

### Grad 3: Vernetzt (Future)

**Ziel**: Multiplayer-Support und Community-Features.

#### Features

| Feature | Beschreibung | Priorität |
|---------|-------------|-----------|
| Multiplayer Sync | Echtzeit-Sync über Server | Future |
| Server-Backend | REST API + Datenbank | Future |
| User-Auth | Anmeldung/Registrierung | Future |
| Playgroups | Gruppen/Clubs verwalten | Future |
| Rankings | ELO oder ähnliches | Future |

---

## UI/UX Überlegungen

### Design-Prinzipien

1. **Mobile-First**: Primär für Smartphones optimiert
2. **Minimal Touch**: Möglichst wenig Tippen, Swipe für Leben
3. **Sofortiges Feedback**: Keine Animationen, instant response
4. **Dark Mode**: Standard, augenschonend
5. **Offline-First**: Funktioniert ohne Internet

### Interaktions-Muster

| Aktion | Eingabe | Feedback |
|--------|---------|----------|
| Leben ändern | Swipe up/down | Vibration + Zahl ändert sich |
| Gift ändern | Tap auf Gift-Button | +1/−1 pro Tap |
| Commander Damage | Tap auf CMD-Bereich | Submenu für Quelle |
| Aktion rückgängig | Tap Undo | Letzte Aktion wird zurückgesetzt |
| Spiel beenden | Tap "Spiel beenden" | Modal mit Gewinner-Auswahl |

### Barrierefreiheit

- Kontrastreiche Farben
- Große Touch-Targets (min 48px)
- Screenreader-Unterstützung für wichtige Aktionen

---

## Technische Architektur

### Projektstruktur

```
mtg-commander-tracker/
├── src/
│   ├── components/         # Vue Komponenten
│   │   ├── PlayerCard.vue
│   │   ├── GameTable.vue
│   │   ├── ActionLog.vue
│   │   ├── SetupScreen.vue
│   │   └── GameEnd.vue
│   ├── composables/       # Wiederverwendbare Logik
│   │   ├── useGameState.ts
│   │   ├── useTimer.ts
│   │   └── useExport.ts
│   ├── stores/           # State Management (Pinia optional)
│   │   └── gameStore.ts
│   ├── services/          # Externe Services
│   │   └── exportService.ts
│   ├── types/            # TypeScript Definitionen
│   │   └── game.ts
│   ├── App.vue
│   └── main.ts
├── public/
│   ├── manifest.json
│   └── icons/
├── index.html
├── vite.config.ts
├── tsconfig.json
└── package.json
```

### State Management

**Option A: Vanilla Vue (empfohlen für Grad 1-2)**
- Reaktive Referenzen (`ref`, `reactive`)
- Einfach, keine额外 Dependencies

**Option B: Pinia (für Grad 3)**
- Falls komplexere State-Logik benötigt

### Daten-Persistenz

```
localStorage
├── game_current     # Aktuelles Spiel (während Spiel)
├── game_history     # Array vergangener Spiele
└── settings         # App-Einstellungen
```

### Export-Service

Der Export-Service formatiert Spieldaten nach dem **MTG Commander Ecosystem** Standard:

```typescript
interface ExportService {
  exportToJSON(game: Game): string;
  exportToCSV(game: Game): string;
  importFromJSON(json: string): Game;
}
```

Siehe: [mtg-commander-ecosystem DATA_SCHEMA](https://github.com/dastuta/mtg-commander-ecosystem)

---

## Entwicklung

### Setup

```bash
# Repository klonen
git clone https://github.com/dastuta/mtg-commander-tracker-v2.git
cd mtg-commander-tracker-v2

# Dependencies installieren
npm install

# Development Server starten
npm run dev

# Production Build
npm run build
```

### Naming Conventions

- **Komponenten**: PascalCase (z.B. `PlayerCard.vue`)
- **Composables**: camelCase mit `use` Prefix (z.B. `useGameState.ts`)
- **Stores**: camelCase (z.B. `gameStore.ts`)
- **CSS-Klassen**: kebab-case (z.B. `.player-card`)

### Code Style

- TypeScript strict mode
- Vue 3 Composition API mit `<script setup>`
- Keine externen CSS-Frameworks (Vanilla CSS)

---

## Offene Fragen

Die folgenden Fragen müssen noch entschieden werden:

### 1. Wie werden Spieler identifiziert?

| Option | Pros | Cons |
|--------|------|------|
| Freitext | Flexibel | Keine Verlaufsdaten |
| Lokale DB | Historie möglich | Mehr Komplexität |
| Server-Login | Cloud-Sync | Grad 3 Feature |

### 2. Commander-Eingabe?

| Option | Pros | Cons |
|--------|------|------|
| Freitext | Einfach | Keine Validierung |
| Scryfall-Autocomplete | Korrekte Namen | Extra API-Call |
| Offline-Liste | Schnell | Veraltet möglich |

### 3. Mehrspieler-Support?

- Sync über lokales Netzwerk?
- Server-basierter Sync?
- Nur lokales Spiel?

### 4. Plattform-Ziel?

- Primär Mobile (iOS/Android PWA)?
- Desktop unterstützend?
- Touch-first vs. Mouse-friendly?

---

## Entscheidungsprotokoll

Entscheidungen werden als ADRs (Architecture Decision Records) im `./docs/ADR/` Verzeichnis dokumentiert.

---

## Verwandte Dokumente

- [MTG Commander Ecosystem](https://github.com/dastuta/mtg-commander-ecosystem) - Datenstandard
- [Daten-Schema](./docs/DATA_SCHEMA.md) - Unser Export-Format

---

## Lizenz

MIT License

## Beitrag

Siehe CONTRIBUTING.md für Richtlinien zur Mitarbeit.
