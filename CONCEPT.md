# MTG Commander Tracker - Konzeptdokument

> **Version**: 0.3.0 (Draft)  
> **Status**: In Planung  
> **Zuletzt aktualisiert**: 2026-03-24

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
  version: string;
  createdAt: string;           // ISO 8601
  
  meta: {
    format: 'commander';
    date: string;
    duration: number;          // Sekunden
    winningPlayerId: string | null;
    winningReason: string | null;
  };
  
  players: Player[];
  
  turns: Turn[];
  
  actions: Action[];           // Alle Aktionen (für Grad 1: nur damage/heal)
  
  finalState: FinalState;
}

interface Player {
  id: string;
  name: string;
  commander: string | null;
  seat: number;
  life: number;                // Start: 40
  isWinner: boolean;
}

interface Turn {
  number: number;
  playerId: string;
  startTime: string;
  endTime: string | null;
}

interface Action {
  id: string;
  turnNumber: number;
  timestamp: string;
  type: 'damage' | 'heal';    // Grad 1: Nur diese beiden
  source: { playerId: string; playerName: string } | null;
  targets: { playerId: string; playerName: string }[];
  value: number;
  previousValue: number;
  resultingValue: number;
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

### Action Logging Konzept

#### Grundprinzip

Jede Aktion im Spiel wird als eigenständiger Eintrag gespeichert. Dies ermöglicht:

1. **Undo/Redo**: Jede Aktion kann rückgängig gemacht werden
2. **Action Log**: Vollständige Historie während des Spiels einsehbar
3. **Export**: Alle Aktionen werden in JSON exportiert

#### Reihenfolge & Zeitstempel

Die Reihenfolge der Aktionen ist **kritisch** für:
- **Undo/Redo**: Letzte Aktion kann mit "Undo" rückgängig gemacht werden
- **Statistik**: Analyse des Spielverlaufs
- **FinalState**: Letzter Zustand beim Spielende

Jede Aktion enthält:
- `timestamp`: ISO 8601 Zeitstempel (eindeutige Sortierung)
- `turnNumber`: Zugnummer (für Statistik)

#### Action Types & Beschreibungen

| Type | Beschreibung | Effekt |
|------|-------------|--------|
| `damage` | Direkter Schaden | Lebenspunkte des Ziels um X reduzieren |
| `heal` | Lebensgewinn | Lebenspunkte des Ziels um X erhöhen |
| `poison` | Giftzähler | Gift-Variable des Ziels um X erhöhen |
| `commander` | Commander-Schaden | Commander-Variable (Paar Quelle+Ziel) um X erhöhen |
| `drain_heal` | Lebensentzug (Zulaport-Style) | Alle Gegner -X Leben, Selbst +X Leben pro Gegner |
| `lifelink_heal` | Lebensdiebstahl | Ziel -X Leben, Selbst +X Leben |
| `counter` | Custom-Zähler | Benutzerdefinierte Variable (Experience, Energy, etc.) |
| `defeat` | Spieler besiegt | **Manueller Button** - setzt Spieler-Status auf "defeated" |
| `victory` | Spiel beendet | **Manueller Button** - beendet das Spiel |

**Hinweis zu `defeat` und `victory`**: Diese sind **ausschließlich manuelle Aktionen** über Buttons. MTG-Karteneffekte können direkt einen Spieler besiegen oder gewinnen lassen - dies wird über diese manuellen Buttons ausgelöst, NICHT automatisch durch Lebenspunkte.

```typescript
type ActionType = 
  | 'damage'        // Direkter Schaden
  | 'heal'          // Heilung
  | 'poison'         // Giftzähler
  | 'commander'     // Commander-Schaden
  | 'drain_heal'    // Lebensentzug
  | 'lifelink_heal' // Lebensdiebstahl
  | 'counter'       // Custom-Zähler
  | 'defeat'        // Manuell: Spieler besiegt
  | 'victory';      // Manuell: Spiel beendet
```

**Geplant für Zukunft (Grad 3+):**
- `token` - Token erstellt/zerstört
- `phase` - Phasen-Änderung
- `roll` - Würfelwurf
- `coin` - Münzwurf

#### Aktions-Struktur

Jede Aktion besteht aus:

| Feld | Beschreibung | Optional |
|------|-------------|----------|
| `id` | Eindeutige ID der Aktion | Nein |
| `turnNumber` | Zugnummer | Nein |
| `timestamp` | Zeitstempel der Aktion | Nein |
| `type` | Art der Aktion (siehe Action Types) | Nein |
| `source` | Quelle der Aktion | Ja |
| `targets` | Ziel/e der Aktion | Nein |
| `value` | Wert der Änderung | Ja |
| `previousValue` | Wert vor der Änderung | Ja |
| `resultingValue` | Wert nach der Änderung | Ja |

#### Quellen (Source)

Die Quelle beschreibt wer die Aktion auslöst:

```typescript
interface ActionSource {
  type: 'player' | 'card' | 'game';
  playerId?: string;     // Spieler-ID
  playerName?: string;   // Spieler-Name
  cardName?: string;     // Kartenname (z.B. "Lightning Bolt")
  description?: string;  // Freitext (z.B. "Commander Damage")
}
```

**Beispiele:**
- Spieler greift an: `{ type: 'player', playerId: 'p1', playerName: 'Alice' }`
- Karte macht Schaden: `{ type: 'card', cardName: 'Lightning Bolt' }`
- Gift von Konfrontation: `{ type: 'card', cardName: 'Tainted Strike' }`
- Game-Ende: `{ type: 'game' }`

#### Ziele (Targets)

Die Ziele beschreiben wen die Aktion betrifft (ein oder mehrere):

```typescript
interface ActionTarget {
  type: 'player' | 'all_players' | 'opponents' | 'self' | 'game';
  playerId?: string;
  playerName?: string;
}
```

**Beispiele:**
- Einzelnes Ziel: `{ type: 'player', playerId: 'p2', playerName: 'Bob' }`
- Alle Spieler: `{ type: 'all_players' }`
- Nur Gegner: `{ type: 'opponents', sourcePlayerId: 'p1' }`
- Selbst: `{ type: 'self', playerId: 'p1' }`

#### Komplexe Aktionen

Manche Aktionen betreffen mehrere Ziele gleichzeitig:

**Drain Heal (Zulaport Blossom-Style):**
```json
{
  "id": "action-123",
  "turnNumber": 5,
  "type": "drain_heal",
  "source": { "type": "player", "playerId": "p1", "playerName": "Alice" },
  "targets": [
    { "type": "opponents", "sourcePlayerId": "p1" },
    { "type": "self", "playerId": "p1" }
  ],
  "value": 3,
  "effects": [
    { "targetPlayerId": "p1", "targetPlayerName": "Alice", "action": "heal", "value": 9 },
    { "targetPlayerId": "p2", "targetPlayerName": "Bob", "action": "damage", "value": 3 },
    { "targetPlayerId": "p3", "targetPlayerName": "Charlie", "action": "damage", "value": 3 }
  ]
}
```

**Commander Damage:**
```json
{
  "id": "action-124",
  "turnNumber": 6,
  "type": "commander",
  "source": { "type": "card", "cardName": "Edgar Markov" },
  "targets": [
    { "type": "player", "playerId": "p2", "playerName": "Bob" }
  ],
  "value": 3,
  "totalDamageFromSource": 9,
  "previousLife": 35,
  "resultingLife": 32
}
```

#### JSON Export Struktur

```typescript
interface GameExport {
  id: string;
  version: string;
  type: 'game';
  createdAt: string;           // ISO 8601
  
  meta: {
    format: 'commander';
    date: string;              // YYYY-MM-DD
    startTime: string;        // ISO 8601
    endTime: string | null;   // ISO 8601
    duration: number;          // Sekunden
    winningPlayerId: string | null;
    winningReason: string | null;
  };
  
  players: Player[];
  
  turns: Turn[];
  
  actions: Action[];
  
  finalState: FinalState;
  
  defeatedPlayers: DefeatedPlayer[];
}
```

#### UI-Konzepte für Actions

**Action Log Anzeige:**
```
┌─────────────────────────────────────┐
│ Zug 5 - Alice                        │
├─────────────────────────────────────┤
│ ⚔️ Alice greift Bob an              │
│    Lightning Bolt → Bob -3 Leben      │
│    (35 → 32)                        │
├─────────────────────────────────────┤
│ 💀 Bob besiegt Charlie               │
│    Grund: Commander Damage            │
│    (Edgar Markov: 21/21)            │
└─────────────────────────────────────┘
```

**Undo-Panel:**
```
┌─────────────────────────────────────┐
│ Verlauf (Undo möglich)              │
├─────────────────────────────────────┤
│ [✓] Zug 5: Alice → Bob -3 Leben    │
│ [✓] Zug 4: Heal +5                 │
│ [✓] Zug 3: Commander 7 Schaden      │
│ [✗] Zug 2: (bereits rückgängig)   │
└─────────────────────────────────────┘
```

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
type ActionType = 
  | 'damage'        // Leben -X
  | 'heal'          // Leben +X
  | 'poison'        // Gift +X
  | 'commander'     // Commander-Schaden +X (Paar Quelle+Ziel)
  | 'drain_heal'   // Alle Gegner -X, Selbst +X pro Gegner
  | 'lifelink_heal'// Ziel -X, Selbst +X
  | 'counter'       // Custom-Zähler (Experience, Energy, etc.)
  | 'defeat'        // Manueller Button: Spieler besiegt
  | 'victory';      // Manueller Button: Spiel beendet

interface Game {
  id: string;
  version: string;
  createdAt: string; // ISO 8601
  
  meta: {
    format: 'commander';
    date: string;
    startTime: string;
    endTime: string | null;
    duration: number;
    winningPlayerId: string | null;
    winningReason: string | null;
  };
  
  players: Player[];
  actions: Action[];
  turns: Turn[];
  
  finalState: FinalState;
  defeatedPlayers: DefeatedPlayer[];
}

interface Player {
  id: string;
  name: string;
  commander: string | null;
  seat: number;
  life: number;       // Aktuelles Leben
  poison: number;      // Giftzähler
  isDefeated: boolean;
  isWinner: boolean;
}

interface ActionSource {
  type: 'player' | 'card' | 'game';
  playerId?: string;
  playerName?: string;
  cardName?: string;
}

interface ActionTarget {
  type: 'player' | 'all_players' | 'opponents' | 'self' | 'game';
  playerId?: string;
  playerName?: string;
}

interface Action {
  id: string;
  turnNumber: number;
  timestamp: string;        // ISO 8601 - für Reihenfolge & Undo/Redo
  type: ActionType;
  source: ActionSource | null;
  targets: ActionTarget[];
  value: number;
  previousValue?: number;    // Leben/Gift vor der Aktion
  resultingValue?: number;   // Leben/Gift nach der Aktion
  
  // Commander-spezifisch
  cardName?: string;        // Kartenname bei Commander-Schaden
  totalDamageFromSource?: number; // Gesamtschaden von dieser Quelle
  
  // Komplexe Aktionen
  effects?: ActionEffect[];  // Bei Multi-Target Aktionen
  
  // System-Aktionen
  reason?: string;          // Grund bei defeat/victory
  eliminatedBy?: string;     // Wer hat besiegt
}

interface ActionEffect {
  targetPlayerId: string;
  targetPlayerName: string;
  action: 'damage' | 'heal' | 'poison' | 'commander';
  value: number;
  previousValue: number;
  resultingValue: number;
}

interface Turn {
  number: number;
  playerId: string;
  playerName: string;
  startTime: string;
  endTime: string | null;
  duration: number; // Sekunden
}

interface FinalState {
  life: Record<string, number>;
  poison: Record<string, number>;
  commanderDamage: Record<string, number>; // "sourceId-targetId" -> damage
}

interface DefeatedPlayer {
  playerId: string;
  playerName: string;
  reason: 'life' | 'poison' | 'commander_damage' | 'concession' | 'other';
  eliminatedBy: string | null;
  turnEliminated: number;
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

## Entscheidungen (nach Reifegrad)

### 1. Spieler-Identifikation

| Reifegrad | Lösung | Beschreibung |
|-----------|--------|-------------|
| **Grad 1** | Freitext | Spieler name frei eingeben |
| **Grad 2** | Lokale DB/Cache | Speicherung für spätere Spiele auf dem Gerät |
| **Grad 3** | Server-Login | Login an selbst-gehostetem Server, Spieler-Daten auslesen |

### 2. Commander-Eingabe

| Reifegrad | Lösung | Beschreibung |
|-----------|--------|-------------|
| **Grad 1** | Freitext | Commander name frei eingeben |
| **Grad 2** | Scryfall-Autocomplete | Offline-Liste mit Autocomplete durch Scryfall validiert |
| **Grad 3** | Server-Login | Decks und Commander vom Server laden |

### 3. Mehrspieler-Support

| Reifegrad | Lösung | Beschreibung |
|-----------|--------|-------------|
| **Grad 1** | Kein Sync | Nur lokales Spiel auf einem Gerät |
| **Grad 2** | Entfallen | App bleibt autark, kein Live-Sync zwischen Geräten |
| **Grad 3** | Server-Export | Export der Spieldaten an Server; Lesen von Spielern/Decks vom Server |

> **Entscheidung**: Die App bleibt grundsätzlich autark. Lediglich der Export von Spieldaten an einen Server und das Auslesen von gespeicherten Spielern/Decks ist von Bedeutung. Kein Live-Multiplayer-Sync.

### 4. Plattform-Ziel

| Reifegrad | Lösung | Beschreibung |
|-----------|--------|-------------|
| **Grad 1** | Browser | Primär Browser-basiert, maximale Kompatibilität |
| **Grad 2** | Browser + Mobile | Optimierung für Smartphones/Tablets |
| **Grad 3** | Native Apps | Optional: Android/iOS App-Entwicklung |

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
