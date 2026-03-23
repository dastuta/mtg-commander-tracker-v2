# ADR-003: Ecosystem-kompatibles JSON-Datenformat

## Status

Proposed

## Kontext

Die App soll Spielerdaten exportieren können, damit sie von anderen Tools (z.B. Statistik-Apps) importiert werden können. Wir wollen keine proprietäres Format, sondern ein offenes, das in ein größeres Ökosystem passt.

## Entscheidung

Wir verwenden das **MTG Commander Ecosystem** JSON-Schema als Export-Format:

```json
{
  "version": "1.0.0",
  "type": "game",
  "id": "uuid",
  "createdAt": "ISO8601",
  
  "meta": {
    "format": "commander",
    "date": "2024-01-15",
    "duration": 5400,
    "winningPlayerId": "uuid",
    "winningReason": "last_standing"
  },
  
  "players": [
    {
      "id": "uuid",
      "name": "Spieler",
      "commander": { "name": "KCommandeur", "scryfallId": "abc" },
      "seat": 1,
      "isWinner": true
    }
  ],
  
  "actions": [],
  
  "finalState": {
    "life": { "uuid": 35 },
    "poison": { "uuid": 0 },
    "commanderDamage": { "source-target": 7 }
  }
}
```

## Begründung

### Offener Standard
- Jeder kann das Format implementieren
- Fördert Interoperabilität

### Schon existierend
- mtg-commander-ecosystem existiert bereits
- Daten-Schema ist dokumentiert

### Erweiterbar
- Neue Felder können hinzugefügt werden
- Abwärtskompatibel durch Versionierung

## Konsequenzen

### Positiv
- Export ist wiederverwendbar
- Community kann Statistik-Tools bauen
- Klare Struktur hilft bei Entwicklung

### Negativ
- Müssen Schema folgen (weniger Flexibilität)
- Import von anderen Formaten erfordert Mapping

## Alternativen

| Alternative | Warum abgelehnt |
|-------------|-----------------|
| Proprietäres JSON | Keine Interoperabilität |
| CSV | Zu simpel für verschachtelte Daten |
| XML | Veraltet, verbose |

## Referenzen

- [MTG Commander Ecosystem](https://github.com/dastuta/mtg-commander-ecosystem)
- [DATA_SCHEMA.md](https://github.com/dastuta/mtg-commander-ecosystem/blob/main/DATA_SCHEMA.md)
