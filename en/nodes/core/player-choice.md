# Player Choice

The PlayerChoice Node pauses the dialogue and presents the player with a list of answer options. Execution waits until a Choice is selected — or an optional timeout expires.

## When should I use it?

- When the player should make a decision that determines the further course of the dialogue.
- For classic RPG dialogues with multiple response options.
- When individual options should only be visible to players with certain tags or attributes (Requirements on the Choice).
- With a timeout when the choice should be time-limited (e.g. an interrogation scene).
- With ChoiceTags when external systems (Analytics, Achievement) should know what kind of response was chosen.

## Properties

| Property | Type | Default | Meaning |
| --- | --- | --- | --- |
| `PromptText` | `FText` | empty | Optional text above the Choices, e.g. `"You respond:"`. |
| `Choices` | Array `UMayDialogueChoice*` | empty | The answer options. Each is a [Choice Sub-Node](../sub-nodes/choice.md). |
| `ChoiceTimeout` | `float` | `0.0` | Timeout in seconds. `0` = no timeout. |
| `TimeoutDefaultChoice` | `int32` | `0` | Index of the Choice automatically selected on timeout. Only active when `ChoiceTimeout > 0`. |

## Sub-Node: Choice

Each option in the `Choices` array is a [Choice Sub-Node](../sub-nodes/choice.md) with its own text, Requirements, and SideEffects. Details on the [Choice page](../sub-nodes/choice.md).

> 📸 **Image placeholder:** `playerchoice-node-graph.png` — PlayerChoice Node in the graph with three Choices.
> *Setup:* PlayerChoice Node with three Choice pills in the body: `"A friend of the king."`, `"That is none of your business."`, `"Password: 'Shadow Door'."` (last one with a Requirement pill `HasTag Story.HeardPassword`). Three Output pins on the right, each connected to a SayLine Node. Input pin connected to a SayLine Node on the left.

> 📸 **Image placeholder:** `playerchoice-details-panel.png` — Details panel of the PlayerChoice Node.
> *Setup:* Select the PlayerChoice Node. In the Details panel: `PromptText = "You respond:"`, `Choices` (array with 3 entries), `ChoiceTimeout = 8.0`, `TimeoutDefaultChoice = 1`.

## Mini Example

```text
[SayLine: Guard | "What do you want?"]
  │
  ▼
[PlayerChoice: PromptText="You respond:"]
  ├─ Choice 0 "Friend"            ──► [SayLine: "You may pass."]  ──► [Exit: Completed]
  ├─ Choice 1 "Give password"     ──► [SayLine: "Welcome."]       ──► [Exit: Completed]
  │    Requirement: HasTag Story.HeardPassword (FailedAndHidden)
  └─ Choice 2 "Silence"          ──► [SayLine: "Get out!"]       ──► [Exit: Failed]
```

> 📸 **Image placeholder:** `playerchoice-example-graph.png` — Complete mini-graph with PlayerChoice.
> *Setup:* `Entry` → `SayLine "What do you want?"` → `PlayerChoice` (3 Choices). Output 0 → `SayLine "You may pass."` → `Exit Completed`. Output 1 → `SayLine "Welcome."` → `Exit Completed`. Output 2 → `SayLine "Get out!"` → `Exit Failed`. All connections visible. Choice 1 shows Requirement pill.

## Common Pitfalls

- **Requirements are re-evaluated on selection**: If a variable changes between display and click, making a Choice unavailable, the click is rejected. This is intentional but can be surprising.
- **Empty `Choices` array**: The dialogue gets stuck in `WaitingForChoice` because there is nothing to choose. Always add at least one Choice.
