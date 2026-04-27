---
description: Wie du den Dialog-Graph liest — Farben, Formen, Sub-Nodes, Choice-Pins.
---

# Graph & visuelle Sprache

Wer einen Dialog-Asset öffnet, sieht zuerst den Graph. Dieser Abschnitt erklärt, was du siehst — damit du einen fremden Dialog sofort lesen kannst, ohne jeden Node einzeln anzuklicken.

> 📸 **Bild-Platzhalter:** `graph-overview-full.png` — Übersichtsaufnahme eines mittleren Dialog-Graphen.
> *Setup:* Asset `DA_Guard_Greeting` geöffnet. Sichtbar von links nach rechts: `Entry` (grüne Kapsel) → `SayLine "Halt! Wer bist du?"` (dunkelrote Title-Bar, Sprecher: Wächter) → `PlayerChoice` (lila, zwei Choice-Pills im Body) → zwei abgehende Leitungen zu je einer `SayLine`-Reaktion → beide enden in `Exit` (rote Kapsel). Kein Comment-Overlay, sauberes horizontales Layout.

## Grundprinzipien auf einen Blick

- **Farbe = Sprecher.** Wer spricht, erkennst du an der Title-Bar-Farbe, nicht am Text.
- **Text ist inline sichtbar.** Du musst keinen Node anklicken, um zu wissen was er sagt.
- **Sub-Nodes sind Pills im Body.** Requirements, Choices und SideEffects erscheinen direkt am übergeordneten Node — keine eigenen Boxen im Graph.
- **Formen markieren Semantik.** Entry/Exit sind Kapseln, Branch ist diamantförmig, SubGraph hat ein Ordner-Icon.

## Anatomie: SayLine-Node

```
┌────────────────────────────────────────────┐
│ [dunkelrot] Wächter                        │  ← Title-Bar in Sprecher-Farbe
├────────────────────────────────────────────┤
│ Halt! Wer bist du?                         │  ← Inline-Text (editierbar per Doppelklick)
│                                            │
│ 🏷 Dialogue.Emotion.Stern                   │  ← Emotion-Tag (falls gesetzt)
│ ⚙ SetVariable: HasMet = true               │  ← SideEffect-Pill
└────────────────────────────────────────────┘
      ●                                    ●
    Input                               Output
```

- **Title-Bar** trägt die Sprecher-Farbe, die im Speakers-Panel des Assets konfiguriert ist.
- **Inline-Text** zeigt die ersten 2–3 Zeilen. Doppelklick auf den Text öffnet ein Inline-Editfeld — kein Details-Panel nötig.
- **Emotion-Tags** erscheinen als kompakte Chips unter dem Text, wenn sie gesetzt sind.
- **SideEffect-Pills** zeigen Inline-Aktionen (z.B. `SetVariable`, `AddTag`) als einzeilige Einträge am unteren Node-Rand.

> 📸 **Bild-Platzhalter:** `sayline-node-annotated.png` — Ein einzelner SayLine-Node mit Beschriftungs-Overlays.
> *Setup:* Asset-Editor, ein einzelner SayLine-Node ist ausgewählt und zentriert. Sprecher: Wächter (dunkelrote Title-Bar). Text: „Halt! Wer bist du?". Emotion-Tag `Dialogue.Emotion.Stern` sichtbar als kleiner blauer Chip. SideEffect-Pill `SetVariable HasMet = true` darunter. Rote Pfeile mit Beschriftungen zeigen auf: Title-Bar („Sprecher-Farbe"), Text („Inline-Text"), Tag-Chip („EmotionTag"), Pill („SideEffect").

## Anatomie: PlayerChoice-Node

```
┌────────────────────────────────────────────┐
│ [lila] Player Choice                       │
├────────────────────────────────────────────┤
│ Du antwortest:                             │  ← PromptText
│                                            │
│ 🔓 Ein Freund des Königs.           ─────► │  ← Choice 0 (frei)
│ 🔒 Ich kenne das Passwort.          ─────► │  ← Choice 1 (geblockt, versteckt)
│ ⚠️ Das geht dich nichts an.         ─────► │  ← Choice 2 (sichtbar, aber gesperrt)
└────────────────────────────────────────────┘
      ●
    Input
```

Jede Choice hat einen **eigenen Output-Pin**. Die Struktur der Entscheidung ist die Struktur des Graphen — du siehst sofort, wie viele Pfade es gibt.

| Icon | Bedeutung im Laufzeit-Preview |
| --- | --- |
| 🔓 Offenes Schloss | Wählbar — Requirement erfüllt oder kein Requirement gesetzt. |
| ⚠️ Gelbes Warnsymbol | Sichtbar, aber nicht wählbar. Tooltip zeigt den Grund. |
| 🔒 Geschlossenes Schloss | Unsichtbar für den Spieler. Wird gefiltert. |

> 📸 **Bild-Platzhalter:** `playerchoice-node-preview.png` — PlayerChoice-Node im Preview-Modus mit Schloss-Icons.
> *Setup:* Preview-Runner aktiv. Der PlayerChoice-Node ist zentriert. Choice 0 zeigt grünes offenes Schloss (keine Anforderung). Choice 1 zeigt rotes Schloss (Requirement fehlgeschlagen, `FailedAndHidden`). Choice 2 zeigt gelbes Warndreieck (Requirement fehlgeschlagen, `FailedButVisible`, ausgegraut). Output-Pins für Choice 0 und 2 sichtbar, Output für Choice 1 gestrichelt.

## Branch vs. PlayerChoice

Beide Nodes werten Bedingungen aus — aber mit unterschiedlicher Absicht:

| | Branch | PlayerChoice |
| --- | --- | --- |
| Wer entscheidet? | Die Logik (automatisch) | Der Spieler (interaktiv) |
| Aussehen | Kompakte Diamant-Box | Breite Box mit Choice-Pills |
| Pins | True / False / Fallback | Ein Pin pro Choice |
| Typische Frage | „Welchen Weg nimmt der Dialog?" | „Was antwortet der Spieler?" |

> 📸 **Bild-Platzhalter:** `branch-vs-playerchoice.png` — Beide Node-Typen nebeneinander.
> *Setup:* Zwei Nodes nebeneinander. Links: `Branch`-Node (blaue Title-Bar, Diamant-Form, drei Pins: True, False, Fallback, Requirement-Pill im Body). Rechts: `PlayerChoice`-Node (lila, breite Box, drei Choice-Pills mit je einem Output-Pin). Roter Pfeil mit Beschriftung: „Logik entscheidet" (Branch), „Spieler entscheidet" (PlayerChoice).

## Sonderformen: Entry, Exit, SubGraph, Knot

| Node | Form | Funktion |
| --- | --- | --- |
| **Entry** | Grüne Kapsel, nur Output-Pin | Startpunkt des Dialogs. Genau einer pro Asset. Nicht löschbar. |
| **Exit** | Rote Kapsel, nur Input-Pin | Endpunkt. Status: Completed oder Failed. |
| **SubGraph** | Box mit Ordner-Icon | Doppelklick öffnet den eingebetteten Sub-Graph mit Breadcrumb-Pfad oben. |
| **Knot** | Kleiner Kreis | Kabel-Führung. Wird beim Compile aufgelöst — Runtime sieht nur die Direktverbindung. |

## Farb-Referenz

| Node-Typ | Standardfarbe |
| --- | --- |
| Entry | Grün |
| Exit | Rot |
| SayLine | Per-Sprecher (aus Speakers-Panel) |
| PlayerChoice | Lila |
| Branch | Blau |
| Random Line | Gelb |
| Wait | Türkis |
| Link / SubGraph | Orange |
| SetVariable / Event | Violett / Gold |

Alle Farben lassen sich unter **Project Settings → MayDialogue Editor** anpassen.

> 📸 **Bild-Platzhalter:** `color-reference-nodes.png` — Eine Reihe verschiedener Node-Typen mit Farbchips.
> *Setup:* Alle Node-Typen nebeneinander in einer Reihe, jeder leer aber farbig. Von links nach rechts: Entry (grün), Exit (rot), SayLine (rot/orange, Sprecher-Farbe), PlayerChoice (lila), Branch (blau), RandomLine (gelb), Wait (türkis), Link (orange), SetVariable (violett). Keine Verbindungen zwischen den Nodes.

## Kommentare und Auto-Layout

- **Comment-Boxen** (Taste `C` oder Rechtsklick → Add Comment): Farbiger Rahmen um eine Node-Gruppe. Bewegt sich mit, wenn du die Gruppe verschiebst.
- **Auto-Layout-Button** in der Toolbar: Sortiert den Graphen nach Ebenen (Entry = links, Exits = rechts). Nützlich für einen chaotisch gewachsenen Graphen. Ein manueller Feinschliff lohnt sich danach.

## Das Outline-Panel

Wenn der Graph zu groß wird, um nach einer bestimmten Zeile zu suchen: Das **Outline-Panel** listet alle Nodes flach auf — mit Sprecher-Farb-Chip, Text-Vorschau und Typ-Badge. Volltextsuche und Typ-Filter sind eingebaut. Klick auf einen Eintrag springt direkt zum Node im Graph.

> 📸 **Bild-Platzhalter:** `outline-panel.png` — Outline-Panel mit gefüllter Node-Liste.
> *Setup:* Outline-Panel-Tab geöffnet (rechts neben dem Graph). Liste zeigt 8–10 Einträge: jeder Eintrag hat links einen farbigen Chip (Sprecher-Farbe), daneben Sprecher-Name und erste Zeile des Texts (abgekürzt auf 60 Zeichen), rechts ein Typ-Badge (`Say`, `PC`, `Br`, `X` etc.). Suchfeld oben ist leer. Ein Eintrag ist blau markiert (ausgewählt).

## Zusammenfassung

Der Graph ist so gestaltet, dass du ihn **lesen** kannst, ohne jeden Node anzuklicken: Sprecher-Farben geben Orientierung, Inline-Texte zeigen den Inhalt, Sub-Node-Pills zeigen Bedingungen und Aktionen, Formen unterscheiden Semantik. Das Outline-Panel ergänzt den Graph als lineare Textliste.

Weiter: [Instance & Lifecycle](instance-lifecycle.md).
