# MTG Commander Tracker - Konzeptdokument

> **Version**: 0.6.0 (Draft)  
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

#### Datenmodell (Grad 1) - Simplified

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
  
  actions: Action[];           // Alle Aktionen (chronologisch)
  
  finalState: FinalState;
}

interface Player {
  id: string;
  name: string;
  commander: string | null;   // Grad 1: null
  seat: number;
  life: number;                // Start: 40
  poison: number;              // Grad 1: 0
  isDefeated: boolean;
  isWinner: boolean;
}

interface Action {
  id: string;                  // Eindeutige ID (z.B. "a-001")
  turn: number;                // Zugnummer
  source: string;               // Spieler-ID
  target: string;               // Spieler-ID
  type: 'damage' | 'heal';    // Grad 1: Nur diese
  value: number;                // Betrag
}

interface FinalState {
  life: Record<string, number>; // playerId -> life
  poison: Record<string, number>; // playerId -> poison
}
```

#### Reifegrad der Daten

Die App folgt dem Prinzip: **Basis muss funktionieren, mehr Daten sind optional.**

| Reifegrad | Aktionen | Details |
|-----------|----------|---------|
| **Grad 1** | Minimal | Nur `source`, `target`, `type`, `value` |
| **Grad 2** | Erweitert | + `timestamp`, `turns`, `poison`, `commander` |
| **Grad 3** | Vollständig | + `effects[]`, `cardName`, `reason`, Custom-Counter |

**Beispiel für Grad 1:**
```json
{
  "actions": [
    { "id": "a-001", "turn": 1, "source": "p1", "target": "p2", "type": "damage", "value": 5 },
    { "id": "a-002", "turn": 2, "source": "p2", "target": "p1", "type": "heal", "value": 3 }
  ]
}
```

**Beispiel für Grad 2+:**
```json
{
  "actions": [
    { 
      "id": "a-001", 
      "turn": 1, 
      "timestamp": "2024-01-15T14:05:00Z",
      "source": { "playerId": "p1", "playerName": "Alice" },
      "target": { "playerId": "p2", "playerName": "Bob" },
      "type": "damage",
      "value": 5,
      "cardName": "Lightning Bolt"
    }
  ]
}
```
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

1. **Undo/Redo**: Letzte Aktion kann rückgängig gemacht werden
2. **Action Log**: Vollständige Historie während des Spiels einsehbar
3. **Export**: Alle Aktionen werden in JSON exportiert

#### Daten-Reifegrad

Die Komplexität der Aktionsdaten orientiert sich am **App-Reifegrad**:

| Grad | Action-Felder | Beispiel |
|------|---------------|----------|
| **1** | Minimal | `{ turn, source, target, type, value }` |
| **2** | Erweitert | + `id`, `timestamp`, `poison`, `commander` |
| **3** | Vollständig | + `effects[]`, `cardName`, `reason` |

#### Action Types

| Type | Beschreibung | Grad |
|------|-------------|------|
| `damage` | Schaden an Leben | 1 |
| `heal` | Lebensgewinn | 1 |
| `poison` | Giftzähler | 2 |
| `commander` | Commander-Schaden | 2 |
| `drain_heal` | Lebensentzug (Zulaport) | 3 |
| `lifelink_heal` | Lebensdiebstahl | 3 |
| `counter` | Custom-Zähler | 3 |
| `defeat` | Manueller Button: Spieler besiegt | 2 |
| `victory` | Manueller Button: Spiel beendet | 2 |

**Hinweis**: `defeat` und `victory` sind **ausschließlich manuelle Aktionen** über Buttons.
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

## UI Konzept

### Grundprinzip

Die App wird auf einem Tablet in Tischmitte positioniert. Die UI zeigt für jeden Spieler eine **Spielerkachel (Player Card)** mit seinen Lebenspunkten. Das Layout passt sich der Spieleranzahl an.

### Design-Prinzipien

1. **Lesbarkeit**: Jeder Spieler muss seine Kachel lesen können
2. **Einfachheit**: Minimaler Aufwand für Aktionen
3. **Sofortiges Feedback**: Keine Animationen, instant response
4. **Dark Mode**: Standard, augenschonend
5. **Offline-First**: Funktioniert ohne Internet

---

## Spielerkachel (Player Card)

Die Grundkachel enthält:

```
┌─────────────────────────────────┐
│      Commander Name              │
│         (optional)              │
├─────────────────────────────────┤
│                                 │
│            40                   │  ← Lebenspunkte
│         Leben                   │
│                                 │
├─────────────────────────────────┤
│  [Gift: 0]    [CMD: 0/21]     │  ← Statistiken
└─────────────────────────────────┘
```

**Grad 1 Kachel:**
```
┌─────────────────────────────────┐
│           Alice                │
│                                 │
│            35                   │
│         Leben                   │
│                                 │
└─────────────────────────────────┘
```

---

## Layout nach Spieleranzahl

### 2 Spieler

```
┌─────────────────────────────────┐
│           Alice                │
│            35                  │
└─────────────────────────────────┘
┌─────────────────────────────────┐
│            Bob                 │
│            40                  │
└─────────────────────────────────┘
```

### 3 Spieler

```
┌───────────┬─────────────────┐
│           │                  │
│  Alice    │      Bob         │
│   35      │       40         │
│           │                  │
├───────────┼─────────────────┤
│           │                  │
│   ⚠️      │     Charlie      │
│  Diana    │       20         │
│   10      │                  │
└───────────┴─────────────────┘
```

### 4 Spieler (Standard)

```
┌─────────────────────────────────┐
│      Player 4 (kopfüber)        │
│            Diana                │
│            15                   │
├─────────────────────────────────┤
│                                 │
│   Player 1        Player 2      │
│     Alice            Bob        │
│      35               28        │
│                                 │
├─────────────────────────────────┤
│      Player 3 (kopfüber)        │
│           Charlie              │
│            0 ⚠️                 │
└─────────────────────────────────┘
```

**Hinweis**: Kacheln 3 & 4 sind **180° gedreht**, damit Spieler gegenüber ihre Daten lesen können.

### 5+ Spieler

```
┌─────────────────────────────────┬───────────┐
│        Player 4 (kopfüber)      │           │
│            Diana                 │  Player 5 │
│            15                    │   (90°)   │
├─────────────────────────────────┤   Erik    │
│                                  │   22     │
│   Player 1        Player 2       │           │
│     Alice            Bob        │           │
│      35               28        │           │
├─────────────────────────────────┤           │
│        Player 3 (kopfüber)       │           │
│           Charlie               │           │
│            0 ⚠️                │           │
└─────────────────────────────────┴───────────┘
```

**Hinweis**: 5. Spieler-Kachel ist **90° gedreht** und am Rand.

---

## Reifegrad-basierte UI

### Grad 1: Minimal UI

**Ziel**: Einfachster Weg, Leben zu ändern.

#### Spielerkachel
```
┌─────────────────────────────────┐
│           Alice                 │
│                                 │
│            35                   │
│         Leben                   │
│                                 │
│        ▲ +1    ▼ -1            │
└─────────────────────────────────┘
```

#### Layout
- 4 Spieler in 2x2 Grid
- Kacheln 3 & 4 gekippt
- Buttons für +1/-1 Leben

#### Bedienung
1. **Tap auf Kachel** → Ziel wählen (Pop-up)
2. **+1/-1 Buttons** → Leben ändern
3. **Undo Button** → Letzte Aktion rückgängig

#### Screen: Spiel

```
┌─────────────────────────────────────────────────┐
│ [Menu ☰]          Commander          [Export 📤]│
├─────────────────────────────────────────────────┤
│                                                  │
│         ┌─────────────┐  ┌─────────────┐       │
│         │   Player 4   │  │   Player 5   │       │
│         │   (kopfüber) │  │    (90°)     │       │
│         │     Diana     │  │     Erik      │       │
│         │      15      │  │      22      │       │
│         └─────────────┘  └─────────────┘       │
│                                                  │
│   ┌─────────────┐  ┌─────────────┐              │
│   │   Player 1   │  │   Player 2   │              │
│   │    Alice     │  │     Bob      │              │
│   │     35      │  │     28       │              │
│   │  ▲ +1  ▼ -1 │  │  ▲ +1  ▼ -1  │              │
│   └─────────────┘  └─────────────┘              │
│                                                  │
│   ┌──────────────────────────────────────┐     │
│   │   Player 3 (kopfüber)                │     │
│   │        Charlie                        │     │
│   │         0                            │     │
│   │       ▲ +1  ▼ -1                    │     │
│   └──────────────────────────────────────┘     │
│                                                  │
├─────────────────────────────────────────────────┤
│  [◀ Undo]        Zug 7 (Alice)      [Ende ▶]   │
└─────────────────────────────────────────────────┘
```

---

### Grad 2: Erweiterte UI

**Ziel**: Poison, Commander Damage, mehr Statistiken.

#### Spielerkachel
```
┌─────────────────────────────────┐
│       Edgar Markov              │
│           (Commander)           │
├─────────────────────────────────┤
│                                 │
│            35                   │
│         Leben                   │
│                                 │
├─────────────────────────────────┤
│  [Gift: 3]       [CMD: 7/21]  │
│   ▲ +1  ▼ -1       (von Bob)   │
└─────────────────────────────────┘
```

#### Neue Features
- Commander-Name anzeigen
- Poison-Counter mit +1/-1
- Commander-Damage pro Quelle
- Farbcodierung für Spieler

#### Bedienung

**Aktion eingeben (Grad 2):**

1. **Tap auf Spieler-Kachel** → "Wer macht etwas?"
2. **Ziel wählen** → "Wen betrifft es?"
3. **Parameter setzen** → "Was passiert?"

```
┌─────────────────────────────────┐
│  Aktion:                        │
│                                 │
│  Quelle: Alice (du)             │
│                                 │
│  Ziel: [Bob ▼]                  │
│                                 │
│  Typ:  [Leben ▼]               │
│                                 │
│  Wert: [-5] [+5]                │
│                                 │
│        [ABBRECHEN] [BESTÄTIGEN] │
└─────────────────────────────────┘
```

---

### Grad 3: Fortgeschrittene UI

**Ziel**: Swipe-Gesten, komplexe Aktionen, Token.

#### Spielerkachel
```
┌─────────────────────────────────┐
│       Edgar Markov              │
│           (CMD)                 │
├─────────────────────────────────┤
│                                 │
│            35                   │
│         Leben                   │
│                                 │
├─────────────────────────────────┤
│  [Gift: 3]  [Treasure: 5]     │
│  [CMD: 7]   [Experience: 2]   │
└─────────────────────────────────┘
```

#### Swipe-Geste (Grad 3)

**Einfachster Weg für Schaden:**

1. **Swipe von Alice zu Bob** → Schaden an Bob
2. **Swipe-Richtung** → + oder -?
3. **Swipe-Länge** → Wie viel Schaden?

```
Alice ──swipe──▶ Bob
      (Damage)

Alice ◀──swipe── Bob
      (Heal)
```

#### Komplexe Aktionen

**Zulaport-Blossom Style:**

```
┌─────────────────────────────────┐
│  Aktion: Drain Heal            │
│                                 │
│  Quelle: Alice                 │
│  Wert:   [3]                   │
│                                 │
│  Effekt:                        │
│  • Alice: +9 Leben              │
│  • Bob: -3 Leben                │
│  • Charlie: -3 Leben            │
│                                 │
│        [ABBRECHEN] [BESTÄTIGEN] │
└─────────────────────────────────┘
```

---

## Interaktions-Matrix (nach Reifegrad)

| Aktion | Grad 1 | Grad 2 | Grad 3 |
|--------|--------|--------|--------|
| Leben ändern | +1/-1 Buttons | +1/-1 Buttons | Swipe |
| Gift ändern | - | +1/-1 Buttons | Swipe |
| Commander Damage | - | Button → Menu | Long-Swipe |
| Poison an Alle | - | Button → Menu | Swipe nach außen |
| Drain Heal | - | - | Button → Menu |
| Undo | Button | Button | Button + Timeline |
| Spiel beenden | Button | Button + Modal | Button + Modal |

---

## Navigation

### Setup-Screen (Vor dem Spiel)

```
┌─────────────────────────────────────────────────┐
│  Commander Tracker                              │
│  ─────────────────                             │
│                                                  │
│  Spieler:  [4 ▼]                               │
│                                                  │
│  ┌─────────┐ ┌─────────┐                       │
│  │ Alice   │ │ Bob     │                       │
│  │[Edgar▼] │ │[Winota] │                       │
│  └─────────┘ └─────────┘                       │
│  ┌─────────┐ ┌─────────┐                       │
│  │ Charlie │ │ Diana   │                       │
│  │[Atraxa] │ │[Krenko] │                       │
│  └─────────┘ └─────────┘                       │
│                                                  │
│  Startleben: [40 ▼]                            │
│                                                  │
│           [SPIEL STARTEN ▶]                     │
│                                                  │
└─────────────────────────────────────────────────┘
```

### Spiel-Ende-Screen

```
┌─────────────────────────────────────────────────┐
│  Spiel beendet!                                 │
│  ─────────────────                             │
│                                                  │
│  ⭐ Gewinner: Alice                            │
│     (Edgar Markov)                             │
│                                                  │
│  Statistik:                                     │
│  • Dauer: 45:23                               │
│  • Züge: 12                                   │
│  • Aktionen: 47                               │
│                                                  │
│  ┌─────────────────────────────────────────┐  │
│  │ Verlauf anzeigen                        │  │
│  └─────────────────────────────────────────┘  │
│                                                  │
│  [NEUES SPIEL]  [EXPORT JSON]  [STATISTIKEN]  │
│                                                  │
└─────────────────────────────────────────────────┘
```

---

## Barrierefreiheit

- Kontrastreiche Farben (WCAG AA)
- Große Touch-Targets (min 48px)
- Haptic Feedback bei Aktionen
- Optional: Screenreader-Unterstützung

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
