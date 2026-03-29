# MTG Commander Tracker - UI Konzept

> **Version**: 0.6.0  
> **Basierend auf**: [CONCEPT.md](./CONCEPT.md)

---

## Grundprinzip

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
│                                 │
│                                 │
├─────────────────────────────────┤
│[Gift: 0] [CMD: 0/21] [Aktionen] │  ← Werte Variablen/Weitere Aktionen
└─────────────────────────────────┘
```

**Grad 1 Kachel:**
```
┌─────────────────────────────────┐
│           Alice                 │
│                                 │
│            35                   │
│                                 │
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
┌───────────┬────────────────┐
│           │                │
│  Alice    │      Bob       │
│   35      │       40       │
│           │                │
├────────────────────────────┤
│                            │
│          Charlie           │
│             20             │
│                            │
└────────────────────────────┘
```

### 4 Spieler (Standard)

```
┌───────────┬─────────────────┐
│           │                 │
│  Alice    │      Bob        │
│   35      │       40        │
│           │                 │
├───────────┼─────────────────┤
│           │                 │
│    Diana  │     Charlie     │
│   10      │       20        │
│           │                 │
└───────────┴─────────────────┘
....
**Hinweis**: Kacheln 3 & 4 sind **180° gedreht**, damit Spieler gegenüber ihre Daten lesen können.

### 5 Spieler

```
┌───────────┬─────────────────┬───────────────┐
│           │                 │               │
│  Alice    │      Bob        │               │
│   35      │       40        │               │
│           │                 │     Martin    │  
├───────────┼─────────────────┤       20      │    
│           │                 │               │
│    Diana  │     Charlie     │               │
│   10      │       20        │               │
│           │                 │               │
└───────────┴─────────────────┤───────────────┘
....
**Hinweis**: Kacheln 3 & 4 sind **180° gedreht**, damit Spieler gegenüber ihre Daten lesen können.
**Hinweis**: 5. Spieler-Kachel ist **90° gedreht** und am Rand.

---
### 6 Spieler

```
┌───────────────┌───────────┬─────────────────┬───────────────┐
│               │           │                 │               │
│               │  Alice    │      Bob        │               │
│               │   35      │       40        │               │
│      John     │           │                 │     Martin    │  
├        40     ├───────────┼─────────────────┤       20      │    
│               │           │                 │               │
│               │    Diana  │     Charlie     │               │
│               │   10      │       20        │               │
│               │           │                 │               │
└───────────────└───────────┴─────────────────┤───────────────┘
....
**Hinweis**: Kacheln 3 & 4 sind **180° gedreht**, damit Spieler gegenüber ihre Daten lesen können.
**Hinweis**: 5. und 6. Spieler-Kachel ist **90° gedreht** und am Rand.

## Reifegrad-basierte UI

### Grad 1: Minimal UI

**Ziel**: Einfachster Weg, Leben zu ändern.

#### Spielerkachel
```
┌─────────────────────────────────┐
│           Alice                 │
│                                 │
│            35                   │
│                                 │
│                                 │
│                                 │
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
│  Wert: [-5] [+5]               │
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
│  [CMD: 7]   [Experience: 2]    │
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
│  Quelle: Alice                  │
│  Wert:   [3]                   │
│                                 │
│  Effekt:                        │
│  • Alice: +9 Leben             │
│  • Bob: -3 Leben              │
│  • Charlie: -3 Leben          │
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

## Verwandte Dokumente

- [CONCEPT.md](./CONCEPT.md) - Hauptkonzept
- [DATA_SCHEMA.md](./docs/DATA_SCHEMA.md) - Datenformat
