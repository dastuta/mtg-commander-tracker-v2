# MTG Commander Tracker

> Open-source Progressive Web App (PWA) für Magic: The Gathering Commander Spiele.

⚠️ **Status**: Dieses Projekt befindet sich in der **Planungsphase**. Der Code wurde noch nicht geschrieben.

## Was ist das?

Eine App zum Tracken von Commander-Spielen mit Fokus auf:

- **Einfaches Life-Tracking** - Für den Spieltisch
- **Detailliertes Logging** - Jede Aktion wird protokolliert
- **Statistik-Export** - JSON-Format für externe Analyse-Tools

## Reifegrad-Modell

Dieses Projekt folgt einem stufenweisen Entwicklungsansatz:

| Grad | Status | Features |
|------|--------|---------|
| **Grad 1: Prototyp** | 🔴 Geplant | Leben, JSON Export |
| **Grad 2: Produktiv** | ⚪ Geplant | Poison, CMD Damage, Action Log |
| **Grad 3: Vernetzt** | ⚪ Zukunft | Multiplayer, Server-Backend |

Siehe [CONCEPT.md](./CONCEPT.md) für die vollständige Spezifikation.

## Features (Geplant)

### Core
- [x] Konzept-Dokumentation
- [ ] Lebenstracker für 2-6 Spieler
- [ ] Swipe-Gesten für schnelle Eingabe
- [ ] Action Log
- [ ] JSON Export (Ecosystem-kompatibel)

### Commander-Spezifisch
- [ ] Poison Counter
- [ ] Commander Damage (pro Spieler-Paar)
- [ ] 21er-Regel (automatische Niederlage)
- [ ] Zug-Timer

### Statistik
- [ ] Spieler-Datenbank
- [ ] Commander-Verlauf
- [ ] Winrate-Statistiken

## Tech Stack

- **Framework**: Vue 3 (Composition API)
- **Build**: Vite
- **PWA**: vite-plugin-pwa
- **Styling**: Vanilla CSS
- **Sprache**: TypeScript

## Dokumentation

- [Konzept-Dokument](./CONCEPT.md) - Vollständige Spezifikation
- [CONTRIBUTING.md](./CONTRIBUTING.md) - Beitragsrichtlinien

## Mitwirken

Dies ist ein Community-Projekt. Beiträge sind willkommen!

Siehe [CONTRIBUTING.md](./CONTRIBUTING.md) für Details.

## Verwandte Projekte

- [MTG Commander Ecosystem](https://github.com/dastuta/mtg-commander-ecosystem) - Datenstandard und Ökosystem

## Lizenz

MIT License - siehe [LICENSE](./LICENSE)

---

*MTG Commander Tracker ist ein Community-Projekt und ist nicht affiliert mit Wizards of the Coast.*
