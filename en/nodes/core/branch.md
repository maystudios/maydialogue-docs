# Branch

The Branch Node is the automatic conditional switch: no player input, purely based on a condition. It evaluates a Requirement and routes the flow to the True or False path.

## When should I use it?

- When the dialogue should show different lines depending on a player state (e.g. "Has the player been here before?").
- For condition gates that the player does not see — the switch happens invisibly in the background.
- As a switch after a SayLine: a different branch follows depending on an attribute value.
- With `bHasFallback` when a third path is needed for degenerate cases (e.g. missing ASC).

## Properties

| Property | Type | Default | Meaning |
| --- | --- | --- | --- |
| `Condition` | `UMayDialogueRequirement*` (Instanced) | empty | The condition. `Passed` → True path; `FailedButVisible` / `FailedAndHidden` → False path. Not set → always True. |
| `bHasFallback` | `bool` | `false` | Enables a third output pin (`Fallback`) for degenerate cases. |

> 📸 **Image placeholder:** `branch-node-graph.png` — Branch Node with two outputs in the graph.
> *Setup:* Branch Node with `Condition` pill `HasTag Story.Met.Guard`. Three pins on the right: `True` connected to `SayLine "Good to see you again."`, `False` connected to `SayLine "Halt! Who are you?"`. `bHasFallback = false`, Fallback pin not visible. Input pin connected to the Entry Node on the left.

> 📸 **Image placeholder:** `branch-details-panel.png` — Details panel of the Branch Node.
> *Setup:* Select the Branch Node. In the Details panel: `Condition` (Instanced Requirement of type `UMayDlgRequirement_HasTag`, `RequiredTag = Story.Met.Guard`), `bHasFallback = false`.

## Mini Example

```text
[Entry]
  │
  ▼
[Branch: HasTag "Story.Met.Guard"]
  ├─ True  ──► [SayLine: Guard | "Good to see you again."] ──► [PlayerChoice]
  └─ False ──► [SayLine: Guard | "Halt! Who are you?"]     ──► [PlayerChoice]
```

> 📸 **Image placeholder:** `branch-example-graph.png` — Entry → Branch → two SayLine paths.
> *Setup:* `Entry` → `Branch` (Condition: HasTag Story.Met.Guard) → True path `SayLine "Good to see you again."` and False path `SayLine "Halt!"`. Both SayLines lead into the same `PlayerChoice` Node. All connections visible.

## Common Pitfalls

- **Condition not set**: The Node always takes the True path. The validator does not show an error for this, but the behavior is likely unintentional.
- **Confusing Branch with PlayerChoice**: Branch is for automatic decisions without UI. If the player should make a choice, use [PlayerChoice](player-choice.md).

## Branch vs. PlayerChoice

| | Branch | PlayerChoice |
| --- | --- | --- |
| Who decides? | Engine (automatic) | Player |
| UI display | No | Yes |
| Number of conditions | 1 | 1 per Choice |
| Blocks the dialogue? | No (immediate) | Yes (waits for input) |
