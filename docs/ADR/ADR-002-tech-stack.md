# ADR-002: Vue 3 + TypeScript als Tech Stack

## Status

Proposed

## Kontext

Wir müssen einen Tech Stack für die App wählen. Kriterien:
- Schnelle Entwicklung
- PWA-Unterstützung
- Offline-first fähig
- Gut dokumentiert
- Kleine Community, schnelle Hilfe bei Problemen

## Entscheidung

**Frontend:**
- Vue 3 mit Composition API
- TypeScript (strict mode)
- Vite als Build-Tool

**Styling:**
- Vanilla CSS (kein Framework)
- CSS Custom Properties für Theming

**PWA:**
- vite-plugin-pwa

**State:**
- Vue Reactivity (ref, reactive)
- localStorage für Persistenz

## Begründung

### Vue 3
- Der Nutzer hat bereits Erfahrung mit Vue
- Composition API ist flexibel und testbar
- Gut dokumentiertes Ökosystem

### TypeScript
- Fängt Fehler früh
- Bessere IDE-Unterstützung
- Dient als Dokumentation

### Vanilla CSS
- Keine zusätzliche Lernkurve
- Volle Kontrolle über Styles
- Kleinere Bundle-Size
- Gut für Dark Mode

### Vite
- Schneller Development-Server
- HMR funktioniert tadellos
- Gute PWA-Integration

## Konsequenzen

### Positiv
- Nutzer kann direkt starten
- Kleines Bundle (kein UI-Framework)
- PWA-out-of-the-box

### Negativ
- Mehr CSS manuell schreiben
- Keine vorgefertigten Komponenten

## Alternativen

| Alternative | Warum abgelehnt |
|-------------|-----------------|
| React | Weniger Erfahrung |
| Svelte | Kleineres Ökosystem |
| CSS Framework (Tailwind) | Zu viel Konfiguration |
| State Library (Pinia) | Zu früh, nicht nötig |

## Referenzen

- [Vue 3 Docs](https://vuejs.org/)
- [Vite](https://vitejs.dev/)
- [vite-plugin-pwa](https://vite-pwa-org.netlify.app/)
