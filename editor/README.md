---
description: Der MayDialogue Asset-Editor – welche Panels es gibt und wie du damit arbeitest.
---

# Der Editor

Wenn du ein MayDialogue-Asset im Content Browser doppelklickst, öffnet sich der **Asset-Editor**. Hier entwirfst, testest und debuggst du Dialoge – ohne eine einzige Zeile Code.

> 📸 **Bild-Platzhalter:** `editor-overview-annotated.png` — Gesamtansicht des Asset-Editors mit geöffnetem Beispiel-Dialog.
> *Setup:* Asset `DA_Guard_Greeting` öffnen. Alle Standard-Tabs sichtbar: Graph groß in der Mitte, Details-Panel rechts, Outline links, Compiler-Results unten. Rote Pfeile annotieren: 1) Toolbar mit Save/Compile/Play-Buttons, 2) Graph-Canvas mit drei verbundenen Nodes, 3) Speakers-Panel-Tab-Reiter rechts oben.

## Die Panels im Überblick

| Panel | Wofür | Seite |
| --- | --- | --- |
| **Graph** | Dialog-Struktur bauen – Nodes verbinden, Texte schreiben | [Graph-Panel](graph-panel.md) |
| **Details** | Properties des selektierten Nodes bearbeiten | [Asset-Editor](asset-editor.md) |
| **Speakers** | Sprecher anlegen, Farben und Portraits setzen | [Speakers-Panel](speakers-panel.md) |
| **Variables** | Dialog-Variablen deklarieren und Default-Werte setzen | [Variables-Panel](variables-panel.md) |
| **Outline** | Alle Nodes als durchsuchbare Liste sehen, per Klick anspringen | [Outline](outline.md) |
| **Find Results** | Texte, Tags und Variablen-Namen im ganzen Asset suchen | [Find-in-Dialogue](find.md) |
| **Preview** | Dialog direkt im Editor abspielen – kein PIE nötig | [Preview-Runner](preview-runner.md) |
| **Debugger Watch** | Variablen live beobachten während PIE läuft | [Debugger](debugger.md) |
| **Compiler Results** | Fehler und Warnungen nach dem Compile-Durchlauf | [Asset-Editor](asset-editor.md) |
| **Palette** | Verfügbare Node-Typen per Drag-and-Drop einfügen | [Graph-Panel](graph-panel.md) |

Alle Tabs sind frei andockbar. Dein Layout wird pro Benutzer gespeichert.

## Standard-Layout

```
┌──────────────────────────────────────────────────────────────────┐
│  [Save]  [Compile]  [Auto-Layout]  [Play]  [Find]  [Breakpoint]  │  ← Toolbar
├───────────┬──────────────────────────────────────┬───────────────┤
│           │                                      │               │
│  Outline  │           Graph-Canvas               │    Details    │
│           │                                      │    Speakers   │
│           │                                      │    Variables  │
│           │                                      │    Palette    │
├───────────┴──────────────────────────────────────┴───────────────┤
│        Compiler Results  │  Find Results  │  Debugger Watch      │
└──────────────────────────────────────────────────────────────────┘
```

> 📸 **Bild-Platzhalter:** `editor-default-layout.png` — Standard-Tab-Layout mit leerem neuen Asset.
> *Setup:* Neues `DA_Empty` Asset anlegen und öffnen. Nur der Entry-Node im Graph sichtbar. Tabs in der Sidebar zeigen Details, Speakers, Variables als Reiter.

## Empfohlene Arbeitsreihenfolge

Für einen neuen Dialog von Null:

1. **Asset anlegen** – Rechtsklick im Content Browser → Miscellaneous → MayDialogue Asset.
2. **Sprecher definieren** – Speakers-Panel öffnen, Sprecher hinzufügen, Farbe und DisplayName setzen.
3. **Variablen deklarieren** – Variables-Panel, falls der Dialog Zustände speichern muss.
4. **Graph bauen** – Nodes platzieren, verbinden, Texte inline eingeben.
5. **Compile** – `F7` drücken, Compiler-Results auf Fehler prüfen.
6. **Preview-Runner testen** – Play-Button klicken, Dialog ohne PIE durchspielen.
7. **PIE-Debugger** – Für finale Validierung mit echtem GAS-State im laufenden Spiel.

> 📸 **Bild-Platzhalter:** `editor-workflow-steps.png` — Toolbar-Bereich mit nummerierten roten Pfeilen auf Compile-Button (Schritt 5) und Play-Button (Schritt 6).
> *Setup:* Beliebiges Asset mit zwei SayLine-Nodes offen. Toolbar fokussiert.

## Wann welches Werkzeug

| Frage / Aufgabe | Werkzeug |
| --- | --- |
| Dialog-Ablauf bauen | Graph-Panel |
| Sprecher-Farben und Portraits pflegen | Speakers-Panel |
| Schnelltest einer neuen Zeile | Preview-Runner |
| „Wo steht Satz X?" | Find-in-Dialogue |
| „Wie viele Choices hat der Dialog?" | Outline (Filter: Player Choice) |
| Bug mit echtem Quest-State suchen | PIE-Debugger |
| Unordentlichen Graphen aufräumen | Auto-Layout + Comment-Boxes |

> 📸 **Bild-Platzhalter:** `editor-panel-closeup-speakers.png` — Speakers-Panel ausgeklappt mit zwei Einträgen (Guard, Player), Farb-Chips sichtbar.
> *Setup:* Speakers-Tab aktiv, zwei Sprecher `Dialogue.Speaker.Guard` (dunkelrot) und `Dialogue.Speaker.Player` (grau) eingetragen, DisplayNames und Portrait-Slots sichtbar.

> 📸 **Bild-Platzhalter:** `editor-panel-closeup-outline.png` — Outline-Tab mit vier Einträgen: Entry, SayLine Guard, PlayerChoice, Exit.
> *Setup:* Kleines Asset mit Entry → SayLine → PlayerChoice → Exit. Outline zeigt Farb-Chips, Text-Previews und Typ-Badges.

Los geht's: [Asset-Editor →](asset-editor.md)
