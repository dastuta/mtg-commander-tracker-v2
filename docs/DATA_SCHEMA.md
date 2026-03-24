# MTG Commander Tracker - Daten-Schema

> **Version**: 0.2.0  
> **Basierend auf**: [MTG Commander Ecosystem](https://github.com/dastuta/mtg-commander-ecosystem)

Dieses Dokument beschreibt das JSON-Format für Spielexporte.

## Übersicht

```
Game
├── meta          # Spiel-Metadaten
├── players       # Spieler-Liste
├── turns         # Zug-Historie
├── actions       # Alle Aktionen (chronologisch)
├── finalState    # Endzustand
└── defeatedPlayers  # Besiegte Spieler
```

---

## Vollständige Struktur

```json
{
  "version": "1.0.0",
  "type": "game",
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "createdAt": "2024-01-15T19:30:00Z",
  
  "meta": {
    "format": "commander",
    "date": "2024-01-15",
    "startTime": "2024-01-15T14:00:00Z",
    "endTime": "2024-01-15T15:00:00Z",
    "duration": 3600,
    "winningPlayerId": "player-1",
    "winningReason": "last_standing",
    "groupId": null,
    "notes": null
  },
  
  "players": [
    { 
      "id": "player-1", 
      "name": "Alice", 
      "commander": { "name": "Emiel the Blessed", "scryfallId": "abc123" },
      "seat": 1,
      "life": 28,
      "poison": 0,
      "isDefeated": false,
      "isWinner": true 
    }
  ],
  
  "turns": [
    { 
      "number": 1, 
      "playerId": "player-1",
      "playerName": "Alice",
      "startTime": "2024-01-15T14:00:00Z", 
      "endTime": "2024-01-15T14:05:00Z",
      "duration": 300
    }
  ],
  
  "actions": [],
  
  "finalState": {
    "life": { "player-1": 28, "player-2": 0 },
    "poison": { "player-1": 0, "player-2": 0 },
    "commanderDamage": { "player-2-player-1": 9 }
  },
  
  "defeatedPlayers": []
}
```

---

## Action Types

| Type | Beschreibung |
|------|--------------|
| `damage` | Direkter Schaden an Leben |
| `heal` | Lebensgewinn |
| `poison` | Giftzähler |
| `commander` | Commander-Schaden |
| `drain_heal` | Lebensentzug (Zulaport-Style) |
| `lifelink_heal` | Lebensdiebstahl |
| `token` | Token erstellt/zerstört |
| `counter` | Zähler (Experience, Energy, etc.) |
| `phase` | Phasen-Änderung |
| `roll` | Würfelwurf |
| `coin` | Münzwurf |
| `defeat` | Spieler besiegt |
| `victory` | Spiel beendet |

---

## Action Strukturen

### Simple Action (Schaden/Heilung)

```json
{
  "id": "action-001",
  "turnNumber": 5,
  "timestamp": "2024-01-15T14:30:00Z",
  "type": "damage",
  "source": { "type": "player", "playerId": "player-1", "playerName": "Alice" },
  "targets": [
    { "type": "player", "playerId": "player-2", "playerName": "Bob" }
  ],
  "value": 5,
  "previousValue": 40,
  "resultingValue": 35
}
```

### Commander Damage

```json
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
type ActionType = 
  | 'damage' | 'heal' | 'poison' | 'commander'
  | 'drain_heal' | 'lifelink_heal'
  | 'token' | 'counter' | 'phase'
  | 'roll' | 'coin'
  | 'defeat' | 'victory';

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
