# Player Choice

Der PlayerChoice-Node pausiert den Dialog und präsentiert dem Spieler eine Liste von Antwort-Optionen. Die Ausführung wartet, bis eine Choice gewählt wird — oder ein optionaler Timeout abläuft.

## Wann setze ich ihn ein?

- Wenn der Spieler eine Entscheidung treffen soll, die den weiteren Dialog-Verlauf bestimmt.
- Für klassische RPG-Dialoge mit mehreren Antwortmöglichkeiten.
- Wenn einzelne Optionen nur für Spieler mit bestimmten Tags oder Attributen sichtbar sein sollen (Requirements auf der Choice).
- Mit Timeout, wenn die Wahl zeitlich begrenzt sein soll (z.B. Verhör-Szene).
- Mit ChoiceTags, wenn externe Systeme (Analytics, Achievement) wissen sollen, welche Art Antwort gewählt wurde.

## Properties

| Property | Typ | Standard | Bedeutung |
| --- | --- | --- | --- |
| `PromptText` | `FText` | leer | Optionaler Text über den Choices, z.B. `"Du antwortest:"`. |
| `Choices` | Array `UMayDialogueChoice*` | leer | Die Antwort-Optionen. Jede ist ein [Choice-Sub-Node](../sub-nodes/choice.md). |
| `ChoiceTimeout` | `float` | `0.0` | Timeout in Sekunden. `0` = kein Timeout. |
| `TimeoutDefaultChoice` | `int32` | `0` | Index der Choice, die bei Timeout automatisch gewählt wird. Nur aktiv wenn `ChoiceTimeout > 0`. |

## Sub-Node: Choice

Jede Option im `Choices`-Array ist ein [Choice-Sub-Node](../sub-nodes/choice.md) mit eigenem Text, Requirements und SideEffects. Details auf der [Choice-Seite](../sub-nodes/choice.md).

> 📸 **Bild-Platzhalter:** `playerchoice-node-graph.png` — PlayerChoice-Node im Graph mit drei Choices.
> *Setup:* PlayerChoice-Node mit drei Choice-Pills im Body: `"Ein Freund des Königs."`, `"Das geht dich nichts an."`, `"Passwort: 'Schattentür'."` (letzte mit Requirement-Pill `HasTag Story.HeardPassword`). Drei Output-Pins rechts verbunden mit je einem SayLine-Node. Input-Pin verbunden mit einem SayLine-Node links.

> 📸 **Bild-Platzhalter:** `playerchoice-details-panel.png` — Details-Panel des PlayerChoice-Nodes.
> *Setup:* PlayerChoice-Node auswählen. Im Details-Panel sichtbar: `PromptText = "Du antwortest:"`, `Choices` (Array mit 3 Einträgen), `ChoiceTimeout = 8.0`, `TimeoutDefaultChoice = 1`.

## Mini-Beispiel

```text
[SayLine: Wächter | "Was willst du?"]
  │
  ▼
[PlayerChoice: PromptText="Du antwortest:"]
  ├─ Choice 0 "Freund"            ──► [SayLine: "Dann passiere."]  ──► [Exit: Completed]
  ├─ Choice 1 "Passwort nennen"   ──► [SayLine: "Willkommen."]     ──► [Exit: Completed]
  │    Requirement: HasTag Story.HeardPassword (FailedAndHidden)
  └─ Choice 2 "Schweigen"        ──► [SayLine: "Verschwinde!"]    ──► [Exit: Failed]
```

> 📸 **Bild-Platzhalter:** `playerchoice-example-graph.png` — Vollständiger Mini-Graph mit PlayerChoice.
> *Setup:* `Entry` → `SayLine "Was willst du?"` → `PlayerChoice` (3 Choices). Output 0 → `SayLine "Dann passiere."` → `Exit Completed`. Output 1 → `SayLine "Willkommen."` → `Exit Completed`. Output 2 → `SayLine "Verschwinde!"` → `Exit Failed`. Alle Verbindungen sichtbar. Choice 1 zeigt Requirement-Pill.

## Häufige Fallstricke

- **Requirements werden bei Auswahl erneut geprüft**: Wenn sich zwischen Anzeige und Klick eine Variable ändert und eine Choice dadurch unavailable wird, wird der Klick abgelehnt. Das ist gewollt, kann aber überraschend sein.
- **Leeres `Choices`-Array**: Der Dialog bleibt in `WaitingForChoice` hängen, weil es nichts zu wählen gibt. Immer mindestens eine Choice eintragen.
