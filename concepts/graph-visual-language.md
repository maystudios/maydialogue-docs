# Graph & visuelle Sprache

Die visuelle Sprache von MayDialogue ist kein Zufall. Jede Farbe, jede Pill, jede Icon-Position folgt einer Idee: **Ein Dialog-Asset soll sich wie ein Drehbuch lesen lassen.** Wer spricht, was sagt er, welche Wahl hat der Spieler, was passiert dann.

## Grundprinzipien

1. **Der Graph ist das Dokument.** Nicht der Details-Panel.
2. **Sprecher-Farben als primärer Anker.** Wer spricht, erkennt man sofort.
3. **Inline-Texte statt Property-Jagd.** Doppelklick bearbeitet direkt im Node.
4. **Sub-Nodes kompakt im Body, nicht als eigene Boxen.** Requirements, Choices, SideEffects leben als Pills.
5. **Formen markieren Semantik.** Entry/Exit sind kompakte Kapseln, Branches sind Diamant-ähnlich, SubGraphs haben Ordner-Icons.

## Anatomie eines Say-Line-Nodes

```
┌──────────────────────────────────────────────┐
│ 🔴 Wächter                                    │ ← Title-Bar in Sprecher-Farbe
├──────────────────────────────────────────────┤
│ Halt! Wer bist du?                           │ ← Inline-Text (Preview)
│                                              │
│ 🏷 Dialogue.Emotion.Stern                     │ ← Emotion-Tags (falls gesetzt)
│                                              │
│ ⚙ +SetVariable: HasMet = true                │ ← SideEffect-Pill
└──────────────────────────────────────────────┘
      ●                                      ●
      │                                      │
    Input                                Output
```

Elemente:

* **Title-Bar-Farbe** = Speaker-NodeColor (aus der Sprecher-Liste des Assets).
* **Breakpoint-Indikator** (kleiner roter Punkt) erscheint rechts oben, wenn ein Breakpoint gesetzt ist.
* **Inline-Text** ist ein Auszug (typisch 2-3 Zeilen). Bei längeren Texten gibt es optional eine Expansion-Funktion.
* **Emotion-Tags** werden als kleine Tag-Pills unter dem Text gezeigt, wenn vorhanden.
* **SideEffect-Pills** hängen als kompakte Zeilen am unteren Body-Rand.

## Anatomie eines Player-Choice-Nodes

```
┌──────────────────────────────────────────────┐
│ 🟣 Player Choice                             │
├──────────────────────────────────────────────┤
│ Du antwortest:                               │ ← PromptText
│                                              │
│ 🔓 Ein Freund des Königs.              ────► │ ← Choice 0 (kein Req)
│ 🔒 Ich kenne das Passwort: ...         ────► │ ← Choice 1 (HasTag req, hidden)
│ ⚠️ Das geht dich nichts an.            ────► │ ← Choice 2 (FailedButVisible)
└──────────────────────────────────────────────┘
      ●
      │
    Input
```

Drei Zustände pro Choice:

| Symbol | Bedeutung |
| --- | --- |
| 🔓 | Passed — Choice ist sichtbar und wählbar. |
| ⚠️ | FailedButVisible — Choice ist sichtbar, aber ausgegraut. Tooltip zeigt `UnavailableReason`. |
| 🔒 | FailedAndHidden — Choice wird gar nicht angezeigt. |

Jede Choice hat einen **eigenen Output-Pin**. Die Struktur der Entscheidung ist die Struktur des Graphen.

## Branch- vs. Player-Choice

Ein Branch sieht **anders aus** als ein PlayerChoice:

* **Branch** ist eine kompakte Diamant-Box mit True/False/(Fallback)-Pins. Die Bedingung ist EIN Requirement.
* **PlayerChoice** ist eine breite Box mit N Choice-Pills im Body und N Output-Pins.

Beides evaluiert Requirements, aber:

* **Branch** fragt: „Welchen Weg soll der **Dialog** nehmen?" (automatisch).
* **PlayerChoice** fragt: „Welchen Weg soll der **Spieler** wählen?" (interaktiv).

## Entry, Exit, Knot, SubGraph

Formale Markierungen, die sich visuell absetzen:

* **Entry** — kompakte grüne Kapsel, nur Output-Pin. Nicht löschbar.
* **Exit** — kompakte rote Kapsel, nur Input-Pin.
* **Knot** — kleiner Kreis, dient nur der Kabel-Führung. Beim Compile aufgelöst.
* **SubGraph** — Box mit Ordner-Icon. Doppelklick öffnet den Sub-Graph mit Breadcrumb-Pfad am oberen Rand.

## Kommentare & Reroute

UE-native Tools, voll integriert:

* **Comment** (C oder Rechtsklick → Add Comment): farbiger Rahmen um eine Node-Gruppe. Folgt beim Verschieben, dient der semantischen Gruppierung.
* **Reroute / Knot**: Doppelklick auf einen Draht zwischen zwei Nodes fügt einen Knot ein. Knots werden beim Compile aufgelöst (`ResolveKnotChain`) – Runtime sieht nur die Direktverbindung.

## Auto-Layout

Die Toolbar hat einen **Auto-Layout-Button**. Dahinter steckt eine Sugiyama-artige Layered-Layout-Implementierung:

1. Nodes werden nach Graph-Ebene sortiert (Entry = 0, alle direkten Nachfolger = 1, usw.).
2. Pro Ebene werden Nodes so angeordnet, dass Draht-Kreuzungen minimiert werden.
3. Knots werden durchgereicht, kollabierte Bereiche bleiben stabil.

Ergebnis ist nicht perfekt – ein manueller Feinschliff lohnt sich bei Show-Case-Assets. Aber ein chaotisch gewachsener Graph wird in Sekunden lesbar.

## Farb-Konventionen (Default)

| Node-Typ | Default-Farbe | Editor-Property |
| --- | --- | --- |
| Entry / Exit | Grün / Rot | `EntryExitColor` (Editor-Settings) |
| SayLine | Per-Speaker | aus `FMayDialogueSpeaker.NodeColor` |
| Player Choice | Lila | `PlayerChoiceColor` |
| Branch | Blau | `BranchColor` |
| Random Line | Gelb | `RandomLineColor` |
| Wait | Türkis | `WaitColor` |
| Link / SubGraph | Orange | `LinkColor` / `SubGraphColor` |
| Camera Focus | Hellblau | `CameraFocusColor` |
| Play Animation | Grün | `AnimationColor` |
| Variable / Event | Violett / Gold | `VariableColor` / `LogicColor` |
| Validation Error | Rot | `ErrorColor` |

Alle Werte lassen sich unter **Project Settings → MayDialogue Editor** anpassen.

## Outline-Panel als Gegenpol

Der Graph ist wunderbar, aber er ist 2D. Wenn du suchst: „Wo ist die SayLine mit dem Text *„…du bist schon wieder hier…"*?", willst du nicht einen Minimap-Fetzen, sondern eine **Liste**.

Das Outline-Panel rendert alle Nodes als flache Liste mit:

* **Sprecher-Farb-Chip** links.
* **Primärtext** (Sprecher-Name oder Node-Typ).
* **Sekundärtext** (gekürzter Dialog-Text, max 60 Zeichen).
* **Typ-Badge** rechts (`Say`, `PC`, `Br`, `Rn`, `Wt`, `Ln`, `Sg`, `Ev`, `Fx`, `SV`, `PS`, `CF`, `CS`, `PA`, `E`, `X`).

Mit Volltextsuche, Typ-Filter und Click-to-Jump.

Details siehe [Editor → Outline](../editor/outline.md).

## Zusammenfassung

Der Graph ist die Hauptarbeitsfläche, er rendert **Sprecher, Text und Sub-Nodes inline**, und er ist durch Farben, Icons und Formen so gestaltet, dass ein Dialog-Asset auch ohne Erklärung lesbar ist. Parallel dazu gibt es das Outline-Panel als zweite Sichtweise, die Sprecher und Zeilen linear listet.

Weiter mit dem Kern der Laufzeit: [Instance & Lifecycle](instance-lifecycle.md).
