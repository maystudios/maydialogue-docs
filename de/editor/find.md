---
description: Texte, Tags, Variablen und Sprecher im gesamten Dialog-Asset suchen und per Klick anspringen.
---

# Find-in-Dialogue

Find-in-Dialogue durchsucht das gesamte Asset nach Texten, Tags, Variablen-Namen und Kommentaren. Das Ergebnis ist eine klickbare Liste – jeder Treffer springt direkt zum betreffenden Node im Graph.

## Öffnen

- Toolbar-Button **Find** klicken
- oder Shortcut `Ctrl+F`

Der **Find Results**-Tab kommt in den Vordergrund.

> 📸 **Bild-Platzhalter:** `find-panel-open.png` — Find-Results-Tab im Vordergrund, Suchfeld leer, Cursor blinkt.
> *Setup:* Asset-Editor offen, Ctrl+F gedrückt. Find-Results-Tab springt nach vorne. Suchfeld fokussiert, Cursor blinkt. Leere Ergebnisliste. Roter Pfeil auf Suchfeld.

## Was wird durchsucht?

| Inhalt | Beispiel |
| --- | --- |
| SayLine-Texte | „Halt! Wer bist du?" |
| Choice-Texte | „Ein Freund des Königs." |
| PlayerChoice-Prompt-Texte | „Du antwortest:" |
| Sprecher-DisplayNames | „Wächter", „Narrator" |
| Variablen-Namen | „AngerLevel", „HasMet" |
| GameplayTags | `Dialogue.Emotion.Scared`, `Story.Quest.Found` |
| Editor-Kommentare | Comment-Box-Titel und per-Node-Kommentare |

Die Suche ist **case-insensitiv** und nutzt Substring-Matching: `"keller"` findet `"Der Keller"` und `"Kellerraum"`.

## Ergebnisliste

Pro Treffer eine Zeile:

| Spalte | Inhalt |
| --- | --- |
| **Match-Typ** | Art des Treffers: „SayLine-Text", „Choice-Text", „Speaker", „Tag", „Variable", „Comment" |
| **Kontext** | Treffer im Kontext, der Suchbegriff hervorgehoben |
| **Node** | Klickbare Referenz – Kamera springt im Graph zu diesem Node |

> 📸 **Bild-Platzhalter:** `find-results-list.png` — Find-Results mit vier Treffern für „Keller".
> *Setup:* „Keller" gesucht. Vier Treffer: zwei SayLine-Texte (Match-Typ „SayLine-Text"), ein Comment-Box-Titel (Match-Typ „Comment"), ein Tag `Dialogue.Location.Keller` (Match-Typ „Tag"). Alle Zeilen mit klickbarer Node-Referenz. „Keller" in der Kontext-Spalte fett hervorgehoben. Roter Pfeil auf eine Node-Referenz.

## Suchsyntax

Die Suche ist einfach gehalten: **Substring, keine Regex.** Gib einen oder mehrere Suchbegriffe ein.

Tipps:
- Suche nach einem **Teil eines Tags**: `"Scared"` findet `Dialogue.Emotion.Scared`
- Suche nach einem **Sprecher-Namen**: `"Wächter"` listet alle Nodes, in denen der Wächter spricht
- Suche nach einem **Variablen-Namen**: `"Anger"` findet Nodes die `AngerLevel` setzen oder prüfen
- Suche nach einem **Schlüsselwort**: `"Keller"` zeigt alle Dialog-Zeilen mit diesem Wort

> 📸 **Bild-Platzhalter:** `find-search-tag.png` — Suchfeld mit „Emotion.Scared", Ergebnisliste zeigt drei Nodes mit diesem Tag.
> *Setup:* „Emotion.Scared" in Find eingegeben. Drei Treffer: zwei SayLine-Nodes mit EmotionTag `Dialogue.Emotion.Scared`, ein Branch-Node mit Requirement-Check auf denselben Tag. Match-Typ-Spalte zeigt „Tag". Kontext zeigt den vollständigen Tag-String mit Hervorhebung.

## Typische Einsatzfälle

### Texte vereinheitlichen
Suche nach der alten Schreibweise eines Begriffs. Klick auf jeden Treffer, Text im Details-Panel korrigieren.

### Tag-Nutzung prüfen
Suche nach einem GameplayTag (z.B. `Story.Quest.FoundCodex`) – die Liste zeigt alle Nodes, die ihn setzen oder prüfen. Gut für Impact-Analyse bevor du einen Tag umbenennst.

### Variable auditieren
Suche nach dem Variable-Namen (z.B. `AngerLevel`) – du siehst sofort, welche Nodes ihn lesen (Requirements) und welche schreiben (SetVariable).

### Sprecher-Audit
Suche nach einem Sprecher-DisplayName – du siehst alle Zeilen, die er spricht, und kannst prüfen, ob sein Ton konsistent ist.

### Comment-Boxen finden
Suche nach dem Titel einer Comment-Box, um schnell zu einem kommentierten Abschnitt des Graphs zu springen.

> 📸 **Bild-Platzhalter:** `find-variable-audit.png` — Find-Results für „AngerLevel", Mix aus SayLine-SideEffect-Treffern und Branch-Requirement-Treffern.
> *Setup:* „AngerLevel" gesucht. Vier Treffer: zwei vom Typ „Variable" (SetVariable-Nodes, Kontext zeigt „SetVariable: AngerLevel = 1"), zwei vom Typ „Variable" (CheckDialogueVariable-Requirement, Kontext „AngerLevel >= 3"). Alle vier Zeilen haben klickbare Node-Referenzen.

## Grenzen

- **Nur im aktuellen Asset.** Für projektweite Suche über alle Dialog-Assets nutze den Unreal-eigenen Asset-Finder oder die Reference Viewer.
- **Kein Regex.** Nur Substring-Matching.
- **Nur Textfelder und Tags.** Numerische Property-Werte wie `BlendTime = 1.5` werden nicht gefunden.

{% hint style="info" %}
Wenn du nach einem Tag suchst, das in vielen Assets vorkommt, öffne die Assets nacheinander und nutze jeweils `Ctrl+F`. Es gibt aktuell keine asset-übergreifende Suche im MayDialogue-Editor selbst.
{% endhint %}

Weiter: [Preview-Runner →](preview-runner.md)
