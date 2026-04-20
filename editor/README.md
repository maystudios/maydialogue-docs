# Der Editor

Das `MayDialogueEditor`-Modul ist der Ort, an dem **Designer arbeiten**. Es ist ein vollwertiger Asset-Editor-Toolkit mit zehn Dock-Tabs, eigenem Graph-Schema, integriertem Debugger und In-Editor-Preview.

## Worum es hier geht

In den folgenden Kapiteln findest du:

* **Asset-Editor** – die Tab-Layouts, Toolbars und Commands.
* **Graph-Panel** – die Hauptarbeitsfläche mit Nodes, Pins, Drähten.
* **Speakers-Panel** – Sprecher-Liste des Assets mit Farben, Namen, Portraits.
* **Variables-Panel** – Variable-Definitionen für beide Scopes.
* **Outline** – flache Liste aller Nodes mit Such- und Filter-Funktion.
* **Find-in-Dialogue** – Volltext-Suche über das ganze Asset.
* **Preview-Runner** – Play-Button ohne PIE.
* **Debugger & Breakpoints** – PIE-seitiges Stepping durch Dialoge.
* **Komfort-Features** – Auto-Layout, Knots, Kommentare, Undo/Redo.

## Tab-Übersicht

```
┌───────────────────────────────────────────────────────────────────┐
│  Graph   │  Details  │  Outline  │  Preview  │  Speakers  │       │
├──────────┴───────────┴───────────┴───────────┴────────────┤       │
│                                                           │ Debug │
│                     Graph-Canvas                          │ Watch │
│                                                           │       │
├───────────────────────────────────────────────────────────┤       │
│  Compiler Results   │  Find Results   │  Variables   │ Palette   │
└───────────────────────────────────────────────────────────────────┘
```

Tabs sind frei andockbar. Das Default-Layout optimiert auf: Graph groß in der Mitte, Details rechts, Compiler-Feedback unten.

## Empfohlene Arbeits-Reihenfolge

Für Neueinsteiger:

1. Ein [neues Asset anlegen](asset-editor.md#ein-asset-anlegen).
2. **Sprecher** im [Speakers-Panel](speakers-panel.md) definieren.
3. **Variablen** im [Variables-Panel](variables-panel.md) deklarieren.
4. Den [Graph](graph-panel.md) bauen.
5. **Compile** drücken; Compiler-Results prüfen.
6. Mit dem [Preview-Runner](preview-runner.md) testen.
7. In PIE mit dem [Debugger](debugger.md) validieren.

## Wann welches Werkzeug

| Aufgabe | Werkzeug |
| --- | --- |
| Dialog-Struktur bauen | Graph-Panel |
| Sprecher-Design | Speakers-Panel |
| Schnell-Test ohne PIE | Preview-Runner |
| Bug-Hunting im echten Spiel | PIE-Debugger |
| „Wo steht Text X?" | Find-in-Dialogue |
| „Welche Nodes hat dieses Asset?" | Outline |
| Lesbarkeit verbessern | Auto-Layout / Comments / Knots |

Los geht's: [Asset-Editor →](asset-editor.md).
