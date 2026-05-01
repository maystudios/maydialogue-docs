---
description: Navigation, Nodes platzieren, Verbindungen ziehen, Inline-Edit, Sub-Nodes, Knots und Comments.
---

# Das Graph-Panel

Der Graph ist die Hauptarbeitsfläche. Hier baust du den Ablauf deines Dialogs – von links (Entry) nach rechts (Exit), Node für Node.

> 📸 **Bild-Platzhalter:** `graph-panel-overview.png` — Übersicht eines mittelgroßen Dialogs im Graph.
> *Setup:* Asset `DA_Guard_Greeting` öffnen. Sichtbar: Entry-Node (grün), drei SayLine-Nodes (dunkelrote Titelleisten, Sprecher „Guard"), ein PlayerChoice-Node (blau) mit zwei Choice-Sub-Nodes, zwei weitere SayLines als Reaktion, ein Exit-Node (rot). Alle Nodes verbunden. Kein Zoom – ganzer Graph passt ins Bild.

## Nodes hinzufügen

### Rechtsklick-Menü (schnellster Weg)

Rechtsklick auf eine **leere Graph-Fläche** öffnet das Kontext-Menü. Tippe direkt los – die Liste filtert live nach deiner Eingabe.

Node-Kategorien im Menü:

| Kategorie | Node-Typen |
| --- | --- |
| Dialogue | SayLine, PlayerChoice, RandomLine |
| Flow Control | Branch, Wait, Link, SubGraph |
| Actions | Camera Focus, Camera Shake, Play Animation, Apply Effect, Set Variable, Fire Event, Play Sound, Add Tag, Remove Tag, Trigger Cue |
| Special | Exit, Knot |

> 📸 **Bild-Platzhalter:** `graph-contextmenu.png` — Rechtsklick-Menü auf leerer Graph-Fläche, „SayLine" ins Suchfeld getippt.
> *Setup:* Rechtsklick auf leere Fläche neben Entry-Node. Suchfeld zeigt „SayLine", darunter ein Treffer hervorgehoben. Roter Pfeil auf das Suchfeld.

### Drag-and-Drop aus der Palette

Alternativ: Palette-Tab öffnen, Node-Typ per Drag-and-Drop in den Graph ziehen.

## Nodes verbinden

1. **Klick auf einen Output-Pin** und Maustaste halten.
2. Draht auf den **Input-Pin** des Ziel-Nodes ziehen.
3. Maustaste loslassen – die Verbindung steht.

**Drag auf leere Fläche**: Lässt du den Draht auf einer leeren Stelle los, öffnet sich das Kontext-Menü mit kompatiblen Ziel-Nodes.

{% hint style="info" %}
Der Graph erlaubt nur Verbindungen zwischen kompatiblen Dialog-Pins. Input→Input oder Output→Output funktioniert nicht.
{% endhint %}

## Navigation

### Kamera bewegen (Pan)

| Methode | Aktion |
| --- | --- |
| Rechtsklick + Drag | Graph panen |
| Mittelklick + Drag | Graph panen |
| WASD | Graph panen (nach Klick auf leere Fläche) |

### Zoom

- **Mausrad** zum Rein-/Rauszoomen.
- Aktueller Zoom-Level wird unten rechts angezeigt.

### Schnell-Navigation

| Shortcut | Wirkung |
| --- | --- |
| `F` | Zoom auf selektierte Nodes |
| `Home` | Kamera auf Entry-Node zentrieren |

### SubGraph-Navigation

Doppelklick auf einen SubGraph-Node öffnet den Sub-Graphen. Oben erscheint ein **Breadcrumb-Pfad**:

```
ParentAsset > MainGraph > SubGraphName
```

Klick auf ein Segment springt dorthin zurück.

> 📸 **Bild-Platzhalter:** `graph-breadcrumb.png` — Breadcrumb-Leiste oben im Graph mit zwei Ebenen: „DA_Guard_Greeting > CombatResponse".
> *Setup:* SubGraph-Node doppelklicken. Breadcrumb zeigt Eltern-Asset und Sub-Graph-Name. Roter Pfeil auf Breadcrumb.

## Nodes auswählen und bearbeiten

| Aktion | Wie |
| --- | --- |
| Node auswählen | Einfach-Klick |
| Mehrere auswählen | Shift+Klick oder Rubber-Band (Drag auf leere Fläche) |
| Node verschieben | Auswählen und Drag |
| Node löschen | `Delete` |
| Node duplizieren | `Ctrl+D` |

### Inline-Text-Editing

Bei **SayLine-Nodes**: Doppelklick direkt auf den Textkörper des Nodes öffnet ein Inline-Textfeld. Text eingeben, mit `Enter` oder Fokus-Verlust bestätigen.

Du musst nicht ins Details-Panel wechseln, um Text zu ändern.

> 📸 **Bild-Platzhalter:** `graph-inline-edit.png` — SayLine-Node mit aktivem Inline-Textfeld, Cursor blinkt im Text.
> *Setup:* SayLine-Node doppelklicken. Das Textfeld im Node-Körper ist aktiv, ein blinkender Cursor sichtbar. Roter Pfeil auf das Inline-Textfeld.

## Sub-Nodes hinzufügen

Manche Nodes haben **Sub-Nodes** (Requirements, Choices, SideEffects). Zwei Wege:

**Weg 1 – Rechtsklick auf den Node:**
Rechtsklick auf einen PlayerChoice- oder Branch-Node → *„Add Requirement"* / *„Add SideEffect"* / *„Add Choice"*.

**Weg 2 – Details-Panel:**
1. Node selektieren.
2. Details-Panel → Array `Requirements`, `Choices` oder `SideEffects`.
3. **Add Element** klicken → Typ aus Dropdown wählen.

> 📸 **Bild-Platzhalter:** `graph-subnodes.png` — PlayerChoice-Node mit zwei Choice-Sub-Nodes und einem Requirement-Sub-Node (als Pills am Node-Rand sichtbar).
> *Setup:* PlayerChoice mit zwei Choices „Ein Freund des Königs." und „Das geht dich nichts an." plus einem HasTag-Requirement. Alle Sub-Nodes als kompakte Pills unterhalb des Node-Titels sichtbar.

## Knots (Reroute-Punkte)

Doppelklick auf einen **bestehenden Draht** fügt einen Knot-Punkt ein. Damit kannst du lange Verbindungen sauber um andere Nodes herumführen.

- Knots existieren nur im Editor – der Compiler löst sie vollständig auf.
- Knots unterstützen mehrere Eingänge und Ausgänge.

## Comment-Boxes

Wähle mehrere Nodes aus und drücke `C` → Eine farbige Comment-Box rahmt die Auswahl ein.

- **Titel** und **Farbe** im Details-Panel ändern.
- Verschieben der Box verschiebt alle enthaltenen Nodes mit.
- Nutze Comments für semantische Gruppen: *„Begrüßung"*, *„Konfrontation"*, *„Endings"*.

> 📸 **Bild-Platzhalter:** `graph-comments-knots.png` — Graph mit zwei Comment-Boxen (grün: „Begrüßung", rot: „Konfrontation") und einem Knot-Punkt auf einem langen Draht.
> *Setup:* Drei SayLines in grüner Comment-Box „Begrüßung", zwei weitere in roter Comment-Box „Konfrontation". Verbindungsdraht zwischen den Boxen mit einem Knot oben herumgeführt.

## Copy / Paste (auch Cross-Asset)

| Shortcut | Wirkung |
| --- | --- |
| `Ctrl+C` | Selektierte Nodes kopieren |
| `Ctrl+X` | Ausschneiden |
| `Ctrl+V` | Einfügen |

**Cross-Asset-Paste funktioniert:** Nodes aus einem Dialog-Asset in ein anderes einfügen. Verbindungen innerhalb der kopierten Auswahl bleiben erhalten, alle Nodes bekommen neue GUIDs.

## Breakpoints im Graph setzen

Rechtsklick auf einen Node → **Toggle Breakpoint** (oder `F9` bei selektiertem Node). Ein kleiner roter Punkt erscheint oben rechts am Node.

Breakpoints werden für den [PIE-Debugger](debugger.md) genutzt – die Ausführung pausiert, wenn dieser Node erreicht wird.

> 📸 **Bild-Platzhalter:** `graph-breakpoint.png` — Branch-Node mit rotem Breakpoint-Punkt oben rechts.
> *Setup:* Branch-Node selektiert, F9 gedrückt. Roter Punkt deutlich sichtbar. Roter Annotationspfeil zeigt auf den Punkt mit Beschriftung „Breakpoint".

Weiter: [Speakers-Panel →](speakers-panel.md)
