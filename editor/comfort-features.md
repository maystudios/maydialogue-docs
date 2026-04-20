# Komfort-Features

Kleine Helfer, die große Dialoge handhabbar machen. Dieses Kapitel fasst sie kompakt zusammen.

## Auto-Layout

**Toolbar → Auto-Layout**. Sugiyama-artige Sortierung:

1. Gruppierung nach Graph-Ebene (Entry = 0, nachfolgende Ebenen aufsteigend).
2. Innerhalb einer Ebene: Reihenfolge minimiert Draht-Kreuzungen.
3. Knots werden durchgereicht.

Nicht perfekt – für Show-Case-Assets lohnt manueller Feinschliff. Aber für *„mein Graph ist Chaos"*: Gold wert.

## Comments (Farbige Rahmen)

* **Taste C** mit aktiver Auswahl → Comment-Box um die Nodes.
* **Titel**, **Farbe**, **Textgröße** im Details-Panel.
* Verschieben der Box verschiebt alle enthaltenen Nodes.

Verwende Comments für semantische Gruppen: *„Gesprächsbegrüßung"*, *„Konfrontations-Zweig"*, *„Endings"*.

## Knots (Reroute)

* **Doppelklick auf einen Draht** → Knot eingefügt.
* Knots haben mehrere Ein-/Ausgänge.
* Zur Laufzeit nicht existent – Compiler löst Knot-Ketten auf (`ResolveKnotChain`).

Verwende Knots, um Drähte sauber zu routen, wenn Nodes räumlich weit auseinander liegen.

## Undo / Redo

`Ctrl+Z` / `Ctrl+Y` über alle Graph-Aktionen, inkl. Property-Änderungen, Node-Verschiebungen, Sub-Node-Hinzufügen/-Entfernen.

## Copy / Cut / Paste

* `Ctrl+C` / `Ctrl+X` / `Ctrl+V`.
* **Cross-Asset-Paste**: Nodes aus einem Asset in ein anderes einfügen.
* Beim Paste: neue GUIDs, Verbindungen innerhalb der Auswahl bleiben erhalten.

## Duplicate

`Ctrl+D`. Schnelles Kopieren einer Node-Auswahl mit automatischer Positionierung rechts daneben.

## Speaker-Dropdown

In den Details eines Nodes mit `SpeakerTag`-Property: **kein** generischer Tag-Picker, sondern ein **Dropdown mit allen im Asset definierten Sprechern** (inkl. Farb-Chip + DisplayName).

Das spart deutlich Zeit: du tippst nicht Tag-Pfade, du wählst aus einer kurzen Liste.

## Sub-Graph-Breadcrumbs

In einem Sub-Graph zeigt die Leiste oben den Pfad:

```
ParentAsset > MainGraph > CombatResponse > PainReaction
```

Klick auf ein Segment springt dorthin zurück.

## Drag-and-Drop aus der Palette

Statt Rechtsklick-Menü: zieh einen Node-Typ aus dem **Palette-Tab** direkt in den Graph.

## Editor-Settings

Farben, Auto-Compile, Minimap – alles unter **Project Settings → Plugins → MayDialogue Editor**. Siehe [Editor-Settings](../reference/editor-settings.md).

## Tastatur-Referenz (Zusammenfassung)

| Shortcut | Wirkung |
| --- | --- |
| `Ctrl+S` | Save |
| `F7` | Compile |
| `Ctrl+F` | Find-in-Dialogue |
| `Ctrl+Z` / `Ctrl+Y` | Undo / Redo |
| `Ctrl+C` / `Ctrl+X` / `Ctrl+V` | Copy / Cut / Paste |
| `Ctrl+D` | Duplicate |
| `F9` | Breakpoint togglen |
| `F5` | Continue (Debug) |
| `F10` | Step Over (Debug) |
| `F11` | Step Into (Debug) |
| `Shift+F11` | Step Out (Debug) |
| `C` | Comment-Box um Auswahl |
| `F` | Fit-to-Selection |
| `Home` | Center-on-Entry |

---

Ende des Editor-Kapitels. Weiter mit der **Node-Referenz**: [Core-Nodes →](../nodes/README.md).
