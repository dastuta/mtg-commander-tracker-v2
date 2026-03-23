# Daten-Schema für Export

Dieses Dokument beschreibt das JSON-Format für Spielexporte. Es orientiert sich am [MTG Commander Ecosystem](https://github.com/dastuta/mtg-commander-ecosystem).

## Grad 1: Minimale Struktur

```json
{
  "version": "1.0.0",
  "type": "game",
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "createdAt": "2024-01-15T19:30:00Z",
  
  "meta": {
    "format": "commander",
    "date": "2024-01-15",
    "duration": 3600,
    "winningPlayerId": "player-1",
    "winningReason": "last_standing"
  },
  
  "players": [
    { "id": "player-1", "name": "Alice", "commander": null, "seat": 1, "isWinner": true },
    { "id": "player-2", "name": "Bob", "commander": null, "seat": 2, "isWinner": false },
    { "id": "player-3", "name": "Charlie", "commander": null, "seat": 3, "isWinner": false },
    { "id": "player-4", "name": "Diana", "commander": null, "seat": 4, "isWinner": false }
  ],
  
  "actions": [],
  
  "finalState": {
    "life": {
      "player-1": 35,
      "player-2": 0,
      "player-3": 0,
      "player-4": 0
    },
    "poison": {},
    "commanderDamage": {}
  }
}
```

## Grad 2: Erweiterte Struktur

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
      "isWinner": true 
    },
    { 
      "id": "player-2", 
      "name": "Bob", 
      "commander": { "name": "Winota, Keeper of Force", "scryfallId": "def456" },
      "seat": 2,
      "isWinner": false 
    }
  ],
  
  "turns": [
    { "number": 1, "playerId": "player-1", "startTime": "...", "endTime": "..." },
    { "number": 2, "playerId": "player-2", "startTime": "...", "endTime": "..." }
  ],
  
  "actions": [
    {
      "id": "action-1",
      "turnNumber": 1,
      "timestamp": "2024-01-15T14:05:00Z",
      "type": "damage",
      "category": "life",
      "source": { "playerId": "player-1", "name": "Alice" },
      "target": { "playerId": "player-2", "name": "Bob" },
      "value": 5,
      "previousLife": 40,
      "resultingLife": 35
    },
    {
      "id": "action-2",
      "turnNumber": 3,
      "timestamp": "2024-01-15T14:15:00Z",
      "type": "poison",
      "category": "poison",
      "source": null,
      "target": { "playerId": "player-3", "name": "Charlie" },
      "value": 3,
      "previousPoison": 0,
      "resultingPoison": 3
    },
    {
      "id": "action-3",
      "turnNumber": 5,
      "timestamp": "2024-01-15T14:30:00Z",
      "type": "commander_damage",
      "category": "commander",
      "source": { "playerId": "player-2", "name": "Bob" },
      "target": { "playerId": "player-1", "name": "Alice" },
      "value": 3,
      "sourceCard": "Winota, Keeper of Force",
      "totalDamageFromSource": 9,
      "previousLife": 35,
      "resultingLife": 32
    },
    {
      "id": "action-4",
      "turnNumber": 8,
      "timestamp": "2024-01-15T14:50:00Z",
      "type": "defeat",
      "category": "system",
      "target": { "playerId": "player-3", "name": "Charlie" },
      "reason": "poison",
      "eliminatedBy": null
    }
  ],
  
  "finalState": {
    "life": {
      "player-1": 28,
      "player-2": 15,
      "player-3": 0,
      "player-4": 0
    },
    "poison": {
      "player-1": 0,
      "player-2": 0,
      "player-3": 10,
      "player-4": 0
    },
    "commanderDamage": {
      "player-2-player-1": 9,
      "player-1-player-2": 4
    }
  },
  
  "defeatedPlayers": [
    { 
      "playerId": "player-3", 
      "playerName": "Charlie",
      "reason": "poison",
      "eliminatedBy": null,
      "turnEliminated": 8
    }
  ]
}
```

## Action Types

| Type | Category | Beschreibung |
|------|----------|--------------|
| `damage` | life | Direkter Schaden |
| `heal` | life | Lebensgewinn |
| `poison` | poison | Gift-Zähler |
| `commander_damage` | commander | Commander-Schaden |
| `defeat` | system | Spieler eliminiert |
| `victory` | system | Spieler gewinnt |

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
| `special_effect` | Karten-Effekt (z.B. Thassa's Oracle) |
