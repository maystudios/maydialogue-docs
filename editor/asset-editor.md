---
description: Toolbar, Tabs, Compile-Button – alles, was du über das Asset-Editor-Fenster wissen musst.
---

# Der Asset-Editor

Doppelklick auf ein MayDialogue-Asset im Content Browser öffnet den Asset-Editor. Hier passiert alles: Graph bauen, Compile anstoßen, Preview starten, Fehler lesen.

## Ein Asset anlegen

Drei Wege zum neuen Dialog-Asset:

1. **Content Browser → Rechtsklick → Miscellaneous → MayDialogue Asset**
2. **Add-Button oben links im Content Browser → Miscellaneous → MayDialogue Asset**
3. **Bestehendes Asset duplizieren** (Rechtsklick → Duplicate, dann umbenennen)

Jedes neue Asset startet mit einem Entry-Node und leeren Speakers/Variables-Listen.

> 📸 **Bild-Platzhalter:** `asset-editor-new-asset-context.png` — Rechtsklick-Menü im Content Browser, Maus auf „MayDialogue Asset" zeigend.
> *Setup:* Content Browser in einem leeren Ordner. Rechtsklick-Menü aufgeklappt, Pfad Miscellaneous → MayDialogue Asset mit rotem Pfeil markiert.

## Die Toolbar

Von links nach rechts:

| Button | Shortcut | Was er tut |
| --- | --- | --- |
| **Save** | `Ctrl+S` | Asset speichern |
| **Compile** | `F7` | Validator + Compiler laufen lassen |
| **Auto-Layout** | – | Graph automatisch ordnen (Sugiyama) |
| **Play Dialog** | – | Preview-Runner starten |
| **Find** | `Ctrl+F` | Find-Results-Tab öffnen und fokussieren |
| **Toggle Breakpoint** | `F9` | Breakpoint am selektierten Node setzen/entfernen |

Zusätzlich die Standard-Undo/Redo-Buttons (`Ctrl+Z` / `Ctrl+Y`).

> 📸 **Bild-Platzhalter:** `asset-editor-toolbar-annotated.png` — Toolbar mit nummerierten roten Pfeilen auf jeden Button.
> *Setup:* Asset-Editor offen. Toolbar-Bereich eng beschnitten. Jeder Button hat eine rote Nummer (1=Save, 2=Compile, 3=Auto-Layout, 4=Play, 5=Find, 6=Breakpoint).

## Die Tabs

Der Editor hat **zehn Dock-Tabs**:

| Tab | Inhalt | Standard-Position |
| --- | --- | --- |
| **Graph** | Hauptarbeitsfläche mit Nodes und Verbindungen | Mitte |
| **Details** | Properties des selektierten Nodes | Rechts |
| **Compiler Results** | Fehler und Warnungen nach dem Compile | Unten |
| **Find Results** | Suchergebnisse von Find-in-Dialogue | Unten |
| **Palette** | Alle Node-Typen nach Kategorie | Rechts |
| **Variables** | Variablen-Deklarationen des Assets | Rechts |
| **Speakers** | Sprecher-Liste des Assets | Rechts |
| **Debugger Watch** | Variablen-Watch während PIE | Unten |
| **Preview** | In-Editor Dialog-Preview | Rechts / ausklappbar |
| **Outline** | Alle Nodes als durchsuchbare Liste | Links |

Tabs lassen sich frei verschieben und andocken. Dein Layout bleibt über Sitzungen erhalten.

> 📸 **Bild-Platzhalter:** `asset-editor-tabs-overview.png` — Editor mit geöffnetem Dialog, alle Tab-Reiter sichtbar. Compiler-Results-Tab im Vordergrund mit zwei Warnings.
> *Setup:* Asset `DA_Guard_Patrol` öffnen mit einem absichtlich unverbundenen Exit-Pin, damit der Compiler eine Warning zeigt. Compiler-Results-Tab aktiv zeigen.

## Der Compile-Button

Drücke **Compile** (`F7`) nachdem du den Graph geändert hast. Der Prozess läuft in zwei Schritten:

1. **Validator** prüft auf strukturelle Fehler (fehlende Nodes, unverbundene Pins, leere Texte usw.).
2. Wenn keine Errors vorliegen: **Compiler** übersetzt den Graph in die Runtime-Datenstruktur.

Wenn Errors vorhanden sind, bleibt der letzte korrekte kompilierte Stand aktiv. Du kannst den alten Stand weiter testen, während du die Fehler behebst.

{% hint style="info" %}
**Auto-Compile:** Wenn `Auto Compile on Save` in den MayDialogue Editor-Settings aktiv ist (Standard), läuft der Compiler automatisch beim Speichern. Du musst `F7` dann nur noch manuell drücken, wenn du sofortiges Feedback willst.
{% endhint %}

### Compiler Results – was die Fehlerfarben bedeuten

| Farbe | Bedeutung |
| --- | --- |
| Rot (Error) | Graph wird **nicht** kompiliert, bis der Fehler behoben ist |
| Gelb (Warning) | Graph wird kompiliert, aber es gibt potenzielle Probleme |

Klick auf eine Zeile in der Ergebnisliste springt direkt zum betroffenen Node im Graph.

> 📸 **Bild-Platzhalter:** `asset-editor-compiler-results.png` — Compiler-Results-Tab mit einem Error (rot) und einer Warning (gelb).
> *Setup:* SayLine-Node ohne Text anlegen (Error: „Empty Text") und eine SayLine ohne ausgehenden Pin (Warning: „Unconnected Output"). Compile drücken, Results-Tab zeigen. Roten Pfeil auf die klickbare Node-Referenz in der Error-Zeile.

## Tab: Palette

Die Palette listet alle verfügbaren Node-Typen nach Kategorie:

- **Dialogue:** SayLine, PlayerChoice, RandomLine
- **Flow Control:** Branch, Wait, Link, SubGraph
- **Actions:** Camera Focus, Camera Shake, Play Animation, Apply Effect, Set Variable, Fire Event, Play Sound, Add Tag, Remove Tag, Trigger Cue
- **Special:** Exit, Knot

Drag-and-Drop aus der Palette in den Graph. Alternativ: Rechtsklick auf leere Graph-Fläche öffnet dasselbe Menü.

## Tab: Details

Zeigt die bearbeitbaren Properties des aktuell selektierten Nodes. Bei SayLine-Nodes erscheint beim `SpeakerTag`-Feld kein generischer Tag-Picker, sondern ein **Dropdown mit allen im Asset definierten Sprechern** – inklusive Farb-Chip und DisplayName.

> 📸 **Bild-Platzhalter:** `asset-editor-details-sayline.png` — Details-Panel einer selektierten SayLine.
> *Setup:* SayLine „Halt! Wer bist du?" auswählen. Details-Panel zeigt: SpeakerTag-Dropdown mit Guard-Eintrag (dunkelroter Farb-Chip), DialogueText-Feld gefüllt, AdvanceModeOverride auf Manual, EmotionTags-Array leer. Roter Pfeil auf das Speaker-Dropdown.

## Validator-Checks (Kurzreferenz)

| Prüfung | Stufe | Bedeutung |
| --- | --- | --- |
| Empty Graph | Error | Gar keine Nodes vorhanden |
| Missing Entry | Error | Kein Entry-Node |
| Multiple Entries | Error | Mehr als ein Entry |
| Missing Exit | Warning | Kein Exit-Node (Dialog kann endlos laufen) |
| Unconnected Pins | Error | Output-Pin ohne Verbindung |
| Empty Text | Error | SayLine oder Choice ohne Text |
| Unreachable Nodes | Warning | Nodes die vom Entry aus nicht erreichbar sind |
| Missing Speaker | Error | SayLine ohne gültigen Speaker-Tag |
| Deadlocks | Error | Erreichbarer Node ohne Pfad zum Exit |
| Variable Type Mismatch | Error | Derselbe Variable-Name mit widersprüchlichen Typen |

Weiter: [Graph-Panel →](graph-panel.md)
