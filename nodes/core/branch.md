# Branch

Der Branch-Node ist die automatische Verzweigung: ohne Spieler-Input, rein auf Basis einer Bedingung. Er wertet ein Requirement aus und leitet den Fluss auf den True- oder False-Pfad.

## Wann setze ich ihn ein?

- Wenn der Dialog abhängig von einem Spieler-Zustand unterschiedliche Zeilen zeigen soll (z.B. "War der Spieler schon hier?").
- Für Bedingungs-Gates, die der Spieler nicht sieht — der Wechsel passiert unsichtbar im Hintergrund.
- Als Weiche nach einer SayLine: je nach Attribut-Wert folgt ein anderer Ast.
- Mit `bHasFallback`, wenn ein dritter Pfad für unklare Zustände (z.B. fehlendes ASC) gebraucht wird.

## Properties

| Property | Typ | Standard | Bedeutung |
| --- | --- | --- | --- |
| `Condition` | `UMayDialogueRequirement*` (Instanced) | leer | Die Bedingung. `Passed` → True-Pfad; `FailedButVisible` / `FailedAndHidden` → False-Pfad. Nicht gesetzt → immer True. |
| `bHasFallback` | `bool` | `false` | Aktiviert einen dritten Output-Pin (`Fallback`) für degenerierte Fälle. |

> 📸 **Bild-Platzhalter:** `branch-node-graph.png` — Branch-Node mit zwei Outputs im Graph.
> *Setup:* Branch-Node mit `Condition`-Pill `HasTag Story.Met.Guard`. Drei Pins rechts: `True` verbunden mit `SayLine "Schön, dich wiederzusehen."`, `False` verbunden mit `SayLine "Halt! Wer bist du?"`. `bHasFallback = false`, Fallback-Pin nicht sichtbar. Input-Pin verbunden mit Entry-Node links.

> 📸 **Bild-Platzhalter:** `branch-details-panel.png` — Details-Panel des Branch-Nodes.
> *Setup:* Branch-Node auswählen. Im Details-Panel sichtbar: `Condition` (Instanced Requirement vom Typ `UMayDlgRequirement_HasTag`, `RequiredTag = Story.Met.Guard`), `bHasFallback = false`.

## Mini-Beispiel

```text
[Entry]
  │
  ▼
[Branch: HasTag "Story.Met.Guard"]
  ├─ True  ──► [SayLine: Wächter | "Schön, dich wiederzusehen."] ──► [PlayerChoice]
  └─ False ──► [SayLine: Wächter | "Halt! Wer bist du?"]         ──► [PlayerChoice]
```

> 📸 **Bild-Platzhalter:** `branch-example-graph.png` — Entry → Branch → zwei SayLine-Pfade.
> *Setup:* `Entry` → `Branch` (Condition: HasTag Story.Met.Guard) → True-Pfad `SayLine "Schön, dich wiederzusehen."` und False-Pfad `SayLine "Halt!"`. Beide SayLines führen in denselben `PlayerChoice`-Node. Alle Verbindungen sichtbar.

## Häufige Fallstricke

- **Condition nicht gesetzt**: Der Node nimmt immer den True-Pfad. Der Validator zeigt dafür keinen Error, aber das Verhalten ist wahrscheinlich unbeabsichtigt.
- **Branch verwechseln mit PlayerChoice**: Branch ist für automatische Entscheidungen ohne UI. Wenn der Spieler etwas wählen soll, nimm [PlayerChoice](player-choice.md).

## Branch vs. PlayerChoice

| | Branch | PlayerChoice |
| --- | --- | --- |
| Wer entscheidet? | Engine (automatisch) | Spieler |
| UI-Darstellung | Nein | Ja |
| Anzahl Bedingungen | 1 | 1 pro Choice |
| Blockiert den Dialog? | Nein (immediate) | Ja (wartet auf Eingabe) |
