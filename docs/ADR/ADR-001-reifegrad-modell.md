# ADR-001: Reifegrad-basiertes Entwicklungsmodell

## Status

Proposed

## Kontext

Bei der Entwicklung einer Commander-Tracker-App besteht die Gefahr der Feature-Creep. Ein vollständiges Feature-Set (Poison, Commander Damage, Multiplayer, Statistiken, etc.) kann Monate dauern, bis erste Nutzer die App testen können.

Gleichzeitig ist unklar, welche Features wirklich wichtig sind und welche übersprungen werden können.

## Entscheidung

Wir verwenden ein **Reifegrad-basiertes Entwicklungsmodell**:

1. **Grad 1 (Prototyp)**: Minimal funktionsfähiges Produkt
   - Lebenstracker für 4 Spieler
   - JSON Export
   - Offline-first (localStorage)
   
2. **Grad 2 (Produktiv)**: Vollständiger Commander-Support
   - Poison Counter
   - Commander Damage
   - Action Log
   - Zug-Timer
   
3. **Grad 3 (Vernetzt)**: Multiplayer & Community
   - Server-Backend
   - Echtzeit-Sync
   - User-Auth

## Begründung

- **Schnellere Feedback-Schleifen**: Nutzer können früh testen
- **Klare Prioritäten**: Jeder Grad hat definierte Must-Haves
- **Erweiterbarkeit**: Architektur erlaubt spätere Features ohne Refactoring
- **Risikominimierung**: Falls Projekt eingestellt wird, existiert trotzdem nützlicher Code

## Konsequenzen

### Positiv
- Fokus auf Kern-Features
- Regelmäßige, nutzbare Releases
- Flexibilität bei Feature-Entscheidungen

### Negativ
- Mögliche Architektur-Änderungen zwischen Graden
- Technische Schulden wenn Features übereilt implementiert

## Alternativen

- **Big Bang**: Alles auf einmal entwickeln → Zu lange bis zu nutzbarem Produkt
- **Feature-Branch pro Feature**: Zu viel Overhead für kleines Projekt

## Referenzen

- [Lean Startup](https://de.wikipedia.org/wiki/Lean_Startup)
- [MVP (Minimum Viable Product)](https://de.wikipedia.org/wiki/Minimum_Viable_Product)
