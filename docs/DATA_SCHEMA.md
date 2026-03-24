# MTG Commander Tracker - Daten-Schema

> **Version**: 0.5.0  
> **Basierend auf**: [MTG Commander Ecosystem](https://github.com/dastuta/mtg-commander-ecosystem)

Dieses Dokument beschreibt das JSON-Format für Spielexporte.

## Philosophie: Daten-Reifegrad

Die Komplexität der Daten orientiert sich am **App-Reifegrad**:

- **Grad 1**: Basis-Funktionalität, minimaler Datensatz
- **Grad 2**: Erweiterte Features, mehr Daten
- **Grad 3**: Vollständige Daten für Statistik-Tools

---

## Übersicht

```
Game
├── meta          # Spiel-Metadaten
├── players       # Spieler-Liste
├── actions       # Alle Aktionen (chronologisch)
├── finalState    # Endzustand
└── defeatedPlayers  # Besiegte Spieler
```

---

## Grad 1: Minimal

```json
{
  "version": "1.0.0",
  "type": "game",
  "id": "game-001",
  "createdAt": "2024-01-15T14:00:00Z",
  
  "meta": {
    "format": "commander",
    "date": "2024-01-15",
    "duration": 3600,
    "winningPlayerId": "player-1",
    "winningReason": "last_standing"
  },
  
  "players": [
    { "id": "player-1", "name": "Alice", "seat": 1, "life": 40, "poison": 0, "isDefeated": false, "isWinner": true },
    { "id": "player-2", "name": "Bob", "seat": 2, "life": 0, "poison": 0, "isDefeated": true, "isWinner": false }
  ],
  
  "actions": [
    { "id": "a-001", "turn": 1, "source": "player-1", "target": "player-2", "type": "damage", "value": 5 },
    { "id": "a-002", "turn": 3, "source": "player-2", "target": "player-1", "type": "heal", "value": 3 }
  ],
  
  "finalState": {
    "life": { "player-1": 40, "player-2": 0 },
    "poison": { "player-1": 0, "player-2": 0 }
  },
  
  "defeatedPlayers": []
}
```

**Grad 1 Action:**
```typescript
interface Action {
  id: string;       // Eindeutige ID (z.B. "a-001")
  turn: number;     // Zugnummer
  source: string;   // Spieler-ID
  target: string;   // Spieler-ID
  type: 'damage' | 'heal';
  value: number;    // Betrag
}
```

---

## Grad 2: Erweitert

```json
{
  "version": "1.0.0",
  "type": "game",
  "id": "game-001",
  "createdAt": "2024-01-15T14:00:00Z",
  
  "meta": {
    "format": "commander",
    "date": "2024-01-15",
    "startTime": "2024-01-15T14:00:00Z",
    "endTime": "2024-01-15T15:00:00Z",
    "duration": 3600,
    "winningPlayerId": "player-1",
    "winningReason": "last_standing"
  },
  
  "players": [
    { "id": "player-1", "name": "Alice", "commander": "Edgar Markov", "seat": 1, "life": 35, "poison": 0, "isDefeated": false, "isWinner": true },
    { "id": "player-2", "name": "Bob", "commander": "Winota", "seat": 2, "life": 0, "poison": 0, "isDefeated": true, "isWinner": false }
  ],
  
  "actions": [
    { 
      "id": "a-001", 
      "turn": 1, 
      "timestamp": "2024-01-15T14:05:00Z",
      "source": "player-1", 
      "target": "player-2", 
      "type": "damage", 
      "value": 5,
      "previousLife": 40,
      "resultingLife": 35
    },
    { 
      "id": "a-002", 
      "turn": 4, 
      "timestamp": "2024-01-15T14:20:00Z",
      "source": "player-1", 
      "target": "player-2", 
      "type": "poison", 
      "value": 3,
      "previousPoison": 0,
      "resultingPoison": 3
    },
    { 
      "id": "a-003", 
      "turn": 6, 
      "timestamp": "2024-01-15T14:35:00Z",
      "source": "player-1", 
      "target": "player-2", 
      "type": "commander", 
      "value": 7,
      "cardName": "Edgar Markov",
      "totalDamageFromSource": 14
    }
  ],
  
  "finalState": {
    "life": { "player-1": 35, "player-2": 0 },
    "poison": { "player-1": 0, "player-2": 0 },
    "commanderDamage": { "player-1-player-2": 14 }
  },
  
  "defeatedPlayers": [
    { "playerId": "player-2", "reason": "life", "turnEliminated": 8 }
  ]
}
```

---

## Grad 3: Vollständig

Zusätzlich zu Grad 2:
- Komplexe Aktionen (`drain_heal`, `lifelink_heal`)
- Multi-Target Aktionen mit `effects[]`
- Custom-Counter
- Detaillierte `reason`- und `cardName`-Felder

---

## Action Strukturen

### Grad 1 Action

```json
{ "id": "a-001", "turn": 1, "source": "p1", "target": "p2", "type": "damage", "value": 5 }
```

### Grad 2 Action

```json
{ 
  "id": "a-001", 
  "turn": 5, 
  "timestamp": "2024-01-15T14:30:00Z",
  "source": "p1", 
  "target": "p2", 
  "type": "damage", 
  "value": 5,
  "previousLife": 40,
  "resultingLife": 35
}
```

### Grad 2: Commander Damage

```json
{ 
  "id": "a-003", 
  "turn": 6, 
  "timestamp": "2024-01-15T14:35:00Z",
  "source": "p1", 
  "target": "p2", 
  "type": "commander", 
  "value": 7,
  "cardName": "Edgar Markov",
  "totalDamageFromSource": 14
}
```

### Grad 3: Multi-Target (Drain Heal)

```json
{ 
  "id": "a-010", 
  "turn": 10, 
  "timestamp": "2024-01-15T15:10:00Z",
  "source": "p1", 
  "type": "drain_heal", 
  "value": 3,
  "targets": ["p2", "p3", "p4"],
  "effects": [
    { "target": "p1", "action": "heal", "value": 9 },
    { "target": "p2", "action": "damage", "value": 3 },
    { "target": "p3", "action": "damage", "value": 3 },
    { "target": "p4", "action": "damage", "value": 3 }
  ]
}
```

---

## Action Types (nach Grad)

| Type | Beschreibung | Grad |
|------|-------------|------|
| `damage` | Schaden an Leben | 1 |
| `heal` | Lebensgewinn | 1 |
| `poison` | Giftzähler | 2 |
| `commander` | Commander-Schaden | 2 |
| `drain_heal` | Lebensentzug | 3 |
| `lifelink_heal` | Lebensdiebstahl | 3 |
| `counter` | Custom-Zähler | 3 |
| `defeat` | Manueller Button | 2 |
| `victory` | Manueller Button | 2 |

---

## Defeat Reasons

| Reason | Beschreibung |
|--------|--------------|
| `life` | Leben auf 0 |
| `poison` | Gift ≥ 10 |
| `commander_damage` | CMD-Schaden ≥ 21 |
| `concession` | Spieler gibt auf |
| `other` | Anderer Grund |

## Winning Reasons

| Reason | Beschreibung |
|--------|--------------|
| `last_standing` | Letzter verbleibender |
| `concession` | Alle geben auf |
| `agreed` | Einvernehmlich |
| `special_effect` | Karten-Effekt |

---

## TypeScript Interfaces

### Grad 1

```typescript
interface Action {
  id: string;
  turn: number;
  source: string;    // Spieler-ID
  target: string;   // Spieler-ID
  type: 'damage' | 'heal';
  value: number;
}
```

### Grad 2

```typescript
interface Action {
  id: string;
  turn: number;
  timestamp: string;  // ISO 8601
  source: string;
  target: string;
  type: ActionType;
  value: number;
  
  // Optional: Grad 2+
  previousLife?: number;
  resultingLife?: number;
  cardName?: string;
  totalDamageFromSource?: number;
  reason?: string;
}
```

### Grad 3

```typescript
interface Action {
  id: string;
  turn: number;
  timestamp: string;
  source: string;
  target: string;
  type: ActionType;
  value: number;
  
  // Grad 2+
  previousLife?: number;
  resultingLife?: number;
  cardName?: string;
  totalDamageFromSource?: number;
  
  // Grad 3+
  targets?: string[];        // Multi-Target
  effects?: ActionEffect[];  // Einzelne Effekte
  reason?: string;
}

interface ActionEffect {
  target: string;
  action: 'damage' | 'heal' | 'poison' | 'commander';
  value: number;
}
```
```
{
  "id": "action-002",
  "turnNumber": 6,
  "timestamp": "2024-01-15T14:35:00Z",
  "type": "commander",
  "source": { "type": "card", "cardName": "Edgar Markov" },
  "targets": [
    { "type": "player", "playerId": "player-2", "playerName": "Bob" }
  ],
  "value": 7,
  "totalDamageFromSource": 14,
  "previousValue": 35,
  "resultingValue": 28
}
```

### Poison Counter

```json
{
  "id": "action-003",
  "turnNumber": 8,
  "timestamp": "2024-01-15T14:50:00Z",
  "type": "poison",
  "source": { "type": "card", "cardName": "Tainted Strike" },
  "targets": [
    { "type": "player", "playerId": "player-3", "playerName": "Charlie" }
  ],
  "value": 1,
  "previousValue": 6,
  "resultingValue": 7
}
```

### Group Ping (Alle Gegner)

```json
{
  "id": "action-004",
  "turnNumber": 9,
  "timestamp": "2024-01-15T15:00:00Z",
  "type": "damage",
  "source": { "type": "player", "playerId": "player-1", "playerName": "Alice" },
  "targets": [
    { "type": "opponents", "sourcePlayerId": "player-1" }
  ],
  "value": 2,
  "effects": [
    { "targetPlayerId": "player-2", "targetPlayerName": "Bob", "action": "damage", "value": 2, "previousValue": 28, "resultingValue": 26 },
    { "targetPlayerId": "player-3", "targetPlayerName": "Charlie", "action": "damage", "value": 2, "previousValue": 15, "resultingValue": 13 },
    { "targetPlayerId": "player-4", "targetPlayerName": "Diana", "action": "damage", "value": 2, "previousValue": 8, "resultingValue": 6 }
  ]
}
```

### Drain Heal (Zulaport Style)

```json
{
  "id": "action-005",
  "turnNumber": 10,
  "timestamp": "2024-01-15T15:10:00Z",
  "type": "drain_heal",
  "source": { "type": "player", "playerId": "player-1", "playerName": "Alice" },
  "targets": [
    { "type": "all_players" }
  ],
  "value": 3,
  "effects": [
    { "targetPlayerId": "player-1", "targetPlayerName": "Alice", "action": "heal", "value": 9, "previousValue": 28, "resultingValue": 37 },
    { "targetPlayerId": "player-2", "targetPlayerName": "Bob", "action": "damage", "value": 3, "previousValue": 26, "resultingValue": 23 },
    { "targetPlayerId": "player-3", "targetPlayerName": "Charlie", "action": "damage", "value": 3, "previousValue": 13, "resultingValue": 10 },
    { "targetPlayerId": "player-4", "targetPlayerName": "Diana", "action": "damage", "value": 3, "previousValue": 6, "resultingValue": 3 }
  ]
}
```

### Roll

```json
{
  "id": "action-006",
  "turnNumber": 1,
  "timestamp": "2024-01-15T14:00:00Z",
  "type": "roll",
  "source": { "type": "player", "playerId": "player-1", "playerName": "Alice" },
  "targets": [
    { "type": "game" }
  ],
  "value": 17
}
```

### Defeat

```json
{
  "id": "action-007",
  "turnNumber": 12,
  "timestamp": "2024-01-15T15:25:00Z",
  "type": "defeat",
  "source": { "type": "game" },
  "targets": [
    { "type": "player", "playerId": "player-4", "playerName": "Diana" }
  ],
  "reason": "life",
  "eliminatedBy": null
}
```

### Victory

```json
{
  "id": "action-008",
  "turnNumber": 15,
  "timestamp": "2024-01-15T15:40:00Z",
  "type": "victory",
  "source": { "type": "game" },
  "targets": [
    { "type": "player", "playerId": "player-1", "playerName": "Alice" }
  ],
  "reason": "last_standing"
}
```

---

## Source Types

| Type | Beschreibung | Felder |
|------|--------------|--------|
| `player` | Spieler-initiiert | `playerId`, `playerName` |
| `card` | Von einer Karte | `cardName` |
| `game` | Vom Spiel selbst | - |

## Target Types

| Type | Beschreibung | Felder |
|------|--------------|--------|
| `player` | Einzelner Spieler | `playerId`, `playerName` |
| `all_players` | Alle Spieler | - |
| `opponents` | Alle Gegner eines Spielers | `sourcePlayerId` |
| `self` | Der aktuelle Spieler | `playerId` |
| `game` | Das Spiel selbst | - |

---

## Defeat Reasons

| Reason | Beschreibung |
|--------|--------------|
| `life` | Leben auf 0 |
| `poison` | Gift ≥ 10 |
| `commander_damage` | CMD-Schaden ≥ 21 von einem Spieler |
| `concession` | Spieler gibt auf |
| `other` | Anderer Grund |

## Winning Reasons

| Reason | Beschreibung |
|--------|--------------|
| `last_standing` | Letzter verbleibender Spieler |
| `concession` | Alle Gegner geben auf |
| `agreed` | Einvernehmlich |
| `special_effect` | Karten-Effekt |

---

## Komplexitäts-Level

### Grad 1: Minimal

Nur `damage` und `heal`, einfache Sources/Targets.

```json
{
  "type": "damage",
  "source": { "type": "player", "playerId": "...", "playerName": "..." },
  "targets": [{ "type": "player", "playerId": "...", "playerName": "..." }],
  "value": 5
}
```

### Grad 2: Vollständig

Alle Action Types, Commander Damage, komplexe Aktionen.

---

## TypeScript Interfaces

```typescript
// Grad 1-2: Aktuelle Action Types
type ActionType = 
  | 'damage' | 'heal' | 'poison' | 'commander'
  | 'drain_heal' | 'lifelink_heal'
  | 'counter' | 'defeat' | 'victory';

// Grad 3+: Future Action Types
// | 'token' | 'phase' | 'roll' | 'coin';

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
  timestamp: string;
  type: ActionType;
  source: ActionSource | null;
  targets: ActionTarget[];
  value?: number;
  previousValue?: number;
  resultingValue?: number;
  cardName?: string;
  totalDamageFromSource?: number;
  effects?: ActionEffect[];
  reason?: string;
  eliminatedBy?: string;
}

interface ActionEffect {
  targetPlayerId: string;
  targetPlayerName: string;
  action: 'damage' | 'heal' | 'poison' | 'commander';
  value: number;
  previousValue: number;
  resultingValue: number;
}
```
