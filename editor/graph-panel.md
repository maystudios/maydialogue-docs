# Das Graph-Panel

Der Graph ist die Hauptarbeitsfläche. Dieses Kapitel erklärt, **wie du ihn bedienst** – Nodes platzieren, Pins verbinden, Sub-Nodes anbringen, Ansichten navigieren.

## Bedien-Grundlagen

### Kontext-Menü

**Rechtsklick auf leere Fläche** öffnet das Kontext-Menü. Dort sind alle Node-Typen nach Kategorie gruppiert:

* `Dialogue Nodes`: SayLine, PlayerChoice, RandomLine.
* `Flow Control`: Branch, Wait, Link, SubGraph.
* `Action Nodes`: Camera Focus, Camera Shake, Play Animation, Apply Effect, Set Variable, Fire Event, Play Sound, Add Tag, Remove Tag, Trigger Cue.
* `Special Nodes`: Exit, Knot (Reroute).

Tipp: Tippe direkt los – das Kontext-Menü filtert live nach Namen.

### Drag-and-Drop aus der Palette

Alternativ zum Kontext-Menü: Ziehe einen Node-Typ aus dem **Palette-Tab** direkt in den Graph.

### Pin-Verbindungen

* **Klick auf Output-Pin + Ziehen auf Input-Pin** → Draht entsteht.
* **Ziehen auf leeren Platz** → Kontext-Menü mit kompatiblen Ziel-Nodes erscheint.
* **Doppelklick auf bestehenden Draht** → Knot (Reroute) einfügen.

Der Graph-Schema erlaubt nur Verbindungen zwischen **dialog-kompatiblen Pins** (PC_MayDialogue). Output→Input zwischen zwei Nodes, keine Input→Input oder Output→Output.

### Cycle-Prevention

Der Schema erlaubt Zyklen (z.B. Loop zurück zu einem früheren Node ist erlaubt und für manche Patterns sinnvoll). Der Compiler erkennt Dead-Loops, die keinen Exit erreichen können, und meldet sie als Fehler.

## Node-Interaktionen

### Auswählen

* **Einfach-Klick** wählt einen Node.
* **Shift-Klick** fügt zur Auswahl hinzu.
* **Rubber-Band** (Click + Drag auf leere Fläche) wählt mehrere.

### Inline-Text-Editing

Bei **SayLine-Nodes**: Doppelklick auf den Text im Body startet Inline-Edit. Ein `SEditableTextBox` ersetzt kurzzeitig die Preview. Enter/Escape oder Fokus-Verlust committen.

### Sub-Nodes hinzufügen

Per Details-Panel:

1. Node selektieren.
2. Details-Panel → Array `Requirements` / `Choices` / `SideEffects`.
3. **Add Element** → Typ aus Dropdown wählen.

Oder per Rechtsklick auf den Node direkt im Graph (Kontext-Menü: *„Add Requirement"* / *„Add SideEffect"*).

### Breakpoints

Rechtsklick auf einen Node → **Toggle Breakpoint**. Ein kleiner roter Punkt erscheint rechts-oben am Node. Siehe [Debugger](debugger.md) für die Stepping-Funktionen.

## Navigation

### Pan

* **Rechtsklick + Drag** oder **Mittel-Klick + Drag** panned den Graph.
* **WASD** nach dem Anklicken eines leeren Bereichs.

### Zoom

* **Mausrad**.
* Zoom-Level wird in der Toolbar unten-rechts angezeigt.

### Fit to Window

**F** (bei markierten Nodes) zoomt auf die Auswahl.
**Home** zentriert auf den Entry-Node.

### Breadcrumbs bei SubGraphs

Wenn du auf einen SubGraph-Node doppelklickst, öffnet sich der Sub-Graph mit einem Breadcrumb-Pfad oben (`ParentAsset > SubGraphName > ...`). Klick auf ein Breadcrumb-Segment springt dorthin zurück.

## Kommentare & Reroute

### Kommentare (Comment Boxes)

* **Tastatur C** bei aktiver Auswahl erstellt eine Comment-Box um die ausgewählten Nodes.
* **Details-Panel** ändert Farbe, Titel, Textgröße.
* Verschieben der Comment-Box verschiebt alle enthaltenen Nodes mit.

### Reroute / Knots

* **Doppelklick auf Draht** fügt einen Knot ein.
* Knots dienen nur der **visuellen Kabel-Führung** – sie existieren zur Laufzeit nicht (der Compiler löst Knot-Ketten auf).
* Unterstützen mehrere Ein- und Ausgänge (nützlich für Verteiler).

## Copy/Cut/Paste

* `Ctrl+C` / `Ctrl+X` / `Ctrl+V` über selektierte Nodes.
* **Cross-Asset-Paste**: Du kannst Nodes aus einem Dialog-Asset in ein anderes einfügen. Beim Paste werden neue GUIDs erzeugt, Verbindungen **innerhalb** der kopierten Auswahl bleiben erhalten.

## Selection-Multi-Action

Mit mehreren selektierten Nodes:

* **Delete** entfernt alle.
* **Duplicate** (`Ctrl+D`) kopiert alle.
* **Auto-Layout** (Toolbar-Button) wendet Sugiyama-Layering an.

## Auto-Layout

Die Toolbar-Aktion **Auto-Layout** sortiert den kompletten Graph per Layered-Layout:

1. Gruppierung nach Graph-Tiefe (Entry = Layer 0, direkte Nachfolger = Layer 1, usw.).
2. Reihenfolge innerhalb einer Schicht optimiert Draht-Kreuzungen.
3. Knots werden berücksichtigt.

Nicht perfekt – für Produktions-Assets empfiehlt sich ein manueller Feinschliff. Aber für *„mein Graph ist ein Chaos, bring Ordnung rein"* ist es Gold wert.

## Performance-Hinweise

* Große Graphen (> 200 Nodes) profitieren vom **Outline-Panel** (siehe [Outline](outline.md)) für schnelles Navigieren.
* **Sub-Graphs** helfen, monolithische Graphen zu zerlegen.
* **Comment-Boxen + Grouping** mindern visuelle Belastung.

## Was du merken solltest

* Kontext-Menü ist der schnellste Weg zu neuen Nodes.
* Inline-Text-Edit via Doppelklick.
* Breakpoints per Rechtsklick.
* Knots sind Kabel-Hygiene, Compiler löst sie auf.
* Cross-Asset-Copy-Paste funktioniert.

Weiter: [Speakers-Panel →](speakers-panel.md).
