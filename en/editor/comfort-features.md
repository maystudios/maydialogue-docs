---
description: Auto-Layout, Cross-Asset-Copy/Paste, Undo/Redo, Knots, Comments und alle Shortcuts auf einen Blick.
---

# Komfort-Features

Kleine Helfer, die große Dialoge handhabbar machen. Hier findest du alles, was den Editor schneller und übersichtlicher macht.

## Auto-Layout

**Toolbar → Auto-Layout** oder `Ctrl+L` (falls konfiguriert).

Der Graph wird nach Graph-Tiefe sortiert: Entry bleibt links, direkte Nachfolger eine Ebene weiter rechts, und so weiter. Innerhalb einer Ebene werden Nodes so angeordnet, dass Draht-Kreuzungen minimiert werden.

Wann nutzen:
- Graph ist nach vielen Änderungen unordentlich
- Neuer Mitarbeiter soll den Dialog schnell lesen können
- Vor einem Screenshot für die Doku

Wann manuell nacharbeiten:
- Sehr große Graphen mit vielen parallelen Zweigen
- Comment-Boxen sollen um thematische Gruppen gesetzt werden

> 📸 **Bild-Platzhalter:** `autolayout-before-after.png` — Vorher/Nachher: Links chaotischer Graph, rechts derselbe Graph nach Auto-Layout.
> *Setup:* Zweiteilig. Links: Asset mit 8 Nodes zufällig verteilt, Drähte kreuzen sich mehrfach. Rechts: Dasselbe Asset nach Auto-Layout-Klick – Entry links, Nodes in drei vertikalen Spalten sauber angeordnet, keine Draht-Kreuzungen. Roter Pfeil auf Auto-Layout-Button in der Toolbar (zwischen beiden Varianten).

## Comment-Boxes

Wähle mehrere Nodes aus und drücke `C`. Eine farbige Box rahmt die Auswahl ein.

- **Titel** und **Farbe** im Details-Panel nach Auswahl der Box anpassen.
- **Verschieben** der Box verschiebt alle enthaltenen Nodes mit.
- **Größe** per Drag am Rand ändern.

Sinnvolle Gruppen für Kommentare:
- *„Begrüßung"* – Erster Gesprächsabschnitt
- *„Konfrontation"* – Eskalationszweige
- *„Endings"* – Alle Exit-Pfade

> 📸 **Bild-Platzhalter:** `comfort-comment-boxes.png` — Graph mit zwei farbigen Comment-Boxen und sichtbaren Titeln.
> *Setup:* Graph mit grüner Comment-Box „Begrüßung" (Entry + zwei SayLines), roter Comment-Box „Konfrontation" (PlayerChoice + zwei SayLines + Branch). Beide Boxen mit sichtbaren Titeln oben. Verbindungsdrähte gehen zwischen den Boxen durch.

## Knots (Reroute-Punkte)

Doppelklick auf einen **bestehenden Draht** fügt einen Knot-Punkt ein.

- Knots haben mehrere Ein- und Ausgänge.
- Im Editor: Helfen, lange Drähte um andere Nodes herumzuführen.
- Zur Laufzeit: Existieren nicht – der Compiler löst alle Knot-Ketten auf.

Nutze Knots, wenn:
- Ein Draht andere Nodes verdeckt
- Zwei weit auseinanderliegende Nodes verbunden werden sollen
- Du einen Draht visuell aufteilen willst (ein Ausgang → mehrere Ziele)

> 📸 **Bild-Platzhalter:** `comfort-knots.png` — Zwei Knot-Punkte auf einem langen Draht, der um einen Comment-Block herumgeführt wird.
> *Setup:* SayLine-Node links oben verbunden mit Exit-Node rechts unten, Draht macht einen Bogen um eine Comment-Box in der Mitte. Zwei Knots markieren die Biegepunkte. Roter Pfeil auf einen Knot-Punkt mit Beschriftung „Doppelklick auf Draht".

## Undo / Redo

`Ctrl+Z` / `Ctrl+Y`

Gilt für alle Graph-Aktionen:
- Nodes hinzufügen und löschen
- Verbindungen ziehen und trennen
- Nodes verschieben
- Properties im Details-Panel ändern
- Sub-Nodes hinzufügen und entfernen
- Comment-Boxes erstellen und verschieben

## Copy / Cut / Paste

| Shortcut | Wirkung |
| --- | --- |
| `Ctrl+C` | Selektierte Nodes kopieren |
| `Ctrl+X` | Ausschneiden |
| `Ctrl+V` | Einfügen (mit Offset) |
| `Ctrl+D` | Duplizieren (in einem Schritt, mit automatischem Offset) |

### Cross-Asset-Copy/Paste

Du kannst Nodes aus einem Dialog-Asset kopieren und in ein anderes einfügen:

1. Nodes im Quell-Asset auswählen.
2. `Ctrl+C`.
3. Ziel-Asset öffnen.
4. `Ctrl+V`.

Beim Einfügen bekommen alle Nodes neue GUIDs. Verbindungen **innerhalb** der kopierten Auswahl bleiben erhalten. Verbindungen zu Nodes außerhalb der Auswahl fallen weg.

> 📸 **Bild-Platzhalter:** `comfort-cross-asset-paste.png` — Zwei Asset-Editor-Fenster nebeneinander. Links: Drei selektierte Nodes. Rechts: Dasselbe nach dem Paste, Nodes eingefügt.
> *Setup:* Zwei Editor-Fenster side-by-side. Links `DA_Guard_Day`, drei Nodes markiert (blauer Selektion-Rahmen). Rechts `DA_Guard_Night`, die drei Nodes bereits eingefügt (leicht versetzt, Verbindungen untereinander erhalten). Roter Pfeil zeigt die Kopierrichtung von links nach rechts.

## Speaker-Dropdown im Details-Panel

In den Properties jedes Nodes mit einem `SpeakerTag`-Feld erscheint **kein generischer Tag-Picker**, sondern ein Dropdown mit:
- Farb-Chip des Sprechers
- DisplayName
- Tag als Tooltip

Das spart Tipparbeit und Tippfehler. Sprecher wählen aus einer kurzen Liste statt Tags blind eintippen.

## SubGraph-Breadcrumbs

Wenn du in einem SubGraph-Node doppelklickst, öffnet sich der Sub-Graph. Oben erscheint der Breadcrumb-Pfad:

```
DA_MainDialogue > IntroGraph > CombatResponse
```

Klick auf ein Segment springt dorthin zurück.

## Drag-and-Drop aus der Palette

Statt Rechtsklick → Kontext-Menü: Palette-Tab öffnen, Node-Typ per Drag-and-Drop in den Graph ziehen. Gut, wenn du weißt welchen Typ du willst und das Kontext-Menü-Filtern langsam wird.

## Vollständige Shortcut-Referenz

### Allgemein

| Shortcut | Wirkung |
| --- | --- |
| `Ctrl+S` | Speichern |
| `F7` | Compile |
| `Ctrl+F` | Find-in-Dialogue öffnen |
| `Ctrl+Z` | Undo |
| `Ctrl+Y` | Redo |

### Graph-Bearbeitung

| Shortcut | Wirkung |
| --- | --- |
| `Ctrl+C` | Kopieren |
| `Ctrl+X` | Ausschneiden |
| `Ctrl+V` | Einfügen |
| `Ctrl+D` | Duplizieren |
| `Delete` | Selektierte Nodes löschen |
| `C` | Comment-Box um Auswahl erstellen |

### Navigation

| Shortcut | Wirkung |
| --- | --- |
| `F` | Zoom auf selektierte Nodes |
| `Home` | Auf Entry-Node zentrieren |
| Rechtsklick + Drag | Graph panen |
| Mausrad | Zoom |

### Debugger

| Shortcut | Wirkung |
| --- | --- |
| `F9` | Breakpoint togglen |
| `F5` | Continue |
| `F10` | Step Over |
| `F11` | Step Into |
| `Shift+F11` | Step Out |

> 📸 **Bild-Platzhalter:** `comfort-shortcuts-overview.png` — Shortcut-Cheatsheet als Overlay oder Popup im Editor (falls vorhanden), sonst Toolbar-Bereich mit Tooltips sichtbar.
> *Setup:* Falls der Editor ein Shortcut-Hilfe-Popup hat: Screenshot davon. Alternativ: Toolbar mit geöffnetem Tooltip auf dem Compile-Button, der den F7-Shortcut zeigt.

> 📸 **Bild-Platzhalter:** `comfort-duplicate-workflow.png` — Drei SayLine-Nodes, einer selektiert, nach Ctrl+D erscheint ein Duplikat daneben.
> *Setup:* Mittlerer SayLine-Node selektiert (blauer Rahmen). Nach Ctrl+D: Duplikat erscheint direkt rechts daneben, ebenfalls selektiert, leicht versetzt. Roter Pfeil auf das Duplikat.

---

Ende des Editor-Kapitels. Weiter mit der **Node-Referenz**: [Core-Nodes →](../nodes/README.md)
