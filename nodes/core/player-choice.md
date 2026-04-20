# Player Choice

Der **PlayerChoice-Node** präsentiert dem Spieler eine Liste von Antwort-Optionen.

## Runtime-Verhalten

`ExecuteNode`:

1. Iteriert `Choices[]`.
2. Pro Choice: `Requirements` evaluieren, `FMayDialogueChoiceEntry` erzeugen.
3. `FailedAndHidden`-Choices aus der Liste entfernen.
4. `Instance::PresentChoices(Entries, Timeout, DefaultIndex)` aufrufen.
5. TaskResult: `PauseAndPresentChoices`.

Die Instance wechselt in `WaitingForChoice`. Das Widget erhält `OnChoicesPresented` und zeigt die Buttons.

Bei `SelectChoice(Index)`:

1. Re-Validation der Choice.
2. Sub-Node-`SideEffects` ausführen.
3. Sprung zum `TargetNodeGuid` der gewählten Choice.

## Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `PromptText` | `FText` | Optional; wird über den Choices angezeigt (*„Du antwortest:"*). |
| `Choices` | `TArray<UMayDialogueChoice*>` | Instanced Sub-Node-Array. |
| `ChoiceTimeout` | `float` | 0 = kein Timeout. |
| `TimeoutDefaultChoice` | `int32` | Welcher Choice-Index wird bei Timeout ausgewählt. |

## Sub-Node: Choice

Jede Choice hat:

* `ChoiceText` (FText).
* `ChoiceTags` (GameplayTagContainer – für externe Auswertung).
* `Requirements` (Array).
* `SideEffects` (Array).
* `TargetNodeGuid` (vom Compiler aus Output-Pins befüllt).
* `UnavailableReason` (FText – Tooltip bei FailedButVisible).

Mehr in [Sub-Nodes → Choice](../sub-nodes/choice.md).

## Pins

* **Input**.
* **Mehrere Outputs** – einer pro Choice, in der Graph-Darstellung rechts neben der Choice-Pill.

## Typisches Pattern

```
[SayLine: "Was willst du?"]
  │
  ▼
[PlayerChoice]
  ├─ Choice 0 "Freund"   ──► [SayLine: "Passiere."]
  ├─ Choice 1 "Dieb"     ──► [AddTag-Node] ──► [SayLine: "Verschwinde!"]
  └─ Choice 2 "Passwort" ──► [ApplyEffect-Node] ──► [SayLine: "Willkommen."]
```

## Anmerkungen

* Requirements werden **bei SelectChoice erneut evaluiert** – eine Choice, die in der Wartezeit durch Variable-Änderung unavailable wird, wird beim Klick abgelehnt.
* Das UI zeigt **UnavailableReason** als Tooltip auf ausgegrauten Buttons.
* Der **Skip-Button** hat keine Wirkung im `WaitingForChoice`-Zustand – nur SayLines können geskippt werden.
