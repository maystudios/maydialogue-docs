# Choice

A Choice is an answer option on a [PlayerChoice Node](../core/player-choice.md). It defines the text the player sees, the conditions for its availability, and actions that are executed when it is selected.

## When should I use it?

- Always together with a PlayerChoice Node — each option is a Choice.
- When an answer option should only be available to players with certain tags/attributes (Requirements on the Choice).
- When an action should be triggered immediately on selecting an option, without a separate Action Node (SideEffects on the Choice).
- With `ChoiceTags` when external systems (Analytics, Achievement, UI styling) should know the character of the response.
- With `UnavailableReason` to explain to the player why an option is greyed out.

## Properties

| Property | Type | Default | Meaning |
| --- | --- | --- | --- |
| `ChoiceText` | `FText` | empty | The text of the button the player sees. |
| `ChoiceTags` | `FGameplayTagContainer` | empty | Metadata tags, e.g. `Choice.Aggressive`, `Choice.Peaceful`. For UI styling and external systems. |
| `Requirements` | Array `UMayDialogueRequirement*` | empty | Determine the availability of the Choice (Passed / FailedButVisible / FailedAndHidden). |
| `SideEffects` | Array `UMayDialogueSideEffect*` | empty | Executed when this Choice is selected, before jumping to the next Node. |
| `UnavailableReason` | `FText` | empty | Tooltip text when the Choice is `FailedButVisible`. |
| `TargetNodeGuid` | `FGuid` | (Compiler) | Points to the next Node (from the Output pin). Set by the compiler. |

> 📸 **Image placeholder:** `choice-pill-on-playerchoice.png` — PlayerChoice Node with three Choice pills and an expanded Choice in the Details panel.
> *Setup:* PlayerChoice Node with three Choice pills in the body: `"A friend of the king."` (no Requirements), `"Know the password."` (Requirement pill `HasTag Story.HeardPassword`), `"Bribe with money."` (Requirement pill `CheckAttribute Gold >= 100`, greyed out in preview). Details panel shows Choice 2: `ChoiceText`, `UnavailableReason = "You don't have enough gold."`, Requirements array.

> 📸 **Image placeholder:** `choice-details-panel.png` — Details panel of a single Choice.
> *Setup:* Expand a Choice entry. Visible: `ChoiceText = "Bribe with money."`, `ChoiceTags = (Choice.Bribe)`, `UnavailableReason = "You don't have enough gold."`, `Requirements` (1 entry: CheckAttribute Gold >= 100), `SideEffects` (1 entry: ApplyEffect GE_RemoveGold).

> 📸 **Image placeholder:** `choice-with-sideeffect-pill.png` — Choice pill with SideEffect pill visible in the Node body.
> *Setup:* PlayerChoice Node. Choice pill `"Attack!"` expanded. In the body of this Choice: SideEffect pill `ApplyEffect GE_Adrenaline`. Shows parent Node (PlayerChoice) → Choice pill → SideEffect pill (nested).

## Mini Example

```text
[PlayerChoice: "What do you want to do?"]
  │
  ├─ Choice 0 "Attack!"
  │    ChoiceTags: Choice.Aggressive
  │    SideEffect: ApplyEffect GE_Adrenaline
  │    ──► [SayLine: "Then we fight!"] ──► [Exit: Failed]
  │
  ├─ Choice 1 "Negotiate."
  │    Requirement: HasTag Story.Peaceful (FailedButVisible)
  │    UnavailableReason: "You must be known as peaceful."
  │    ──► [SayLine: "Good, let's talk."] ──► [Exit: Completed]
  │
  └─ Choice 2 "Flee."
       Requirement: CheckAttribute Stamina > 20 (FailedButVisible)
       UnavailableReason: "You are too exhausted."
       ──► [SayLine: "You run away!"] ──► [Exit: Completed]
```

> 📸 **Image placeholder:** `choice-example-graph.png` — Complete PlayerChoice graph with three Choices.
> *Setup:* `Entry` → `SayLine "What do you want to do?"` → `PlayerChoice`. Three Choice pills: `"Attack!"` (SideEffect pill), `"Negotiate."` (Requirement pill, greyed out), `"Flee."` (Requirement pill). Three output paths each to SayLine → Exit.

## Common Pitfalls

- **Requirements are re-evaluated on selection**: If a variable changes between display and click, the Choice may be rejected at SelectChoice even if it was available when displayed.
- **`UnavailableReason` empty for `FailedButVisible`**: The player sees the greyed-out option without an explanation. Always fill `UnavailableReason` when you use `FailResult = FailedButVisible`.

## Extending

{% hint style="success" %}
**Custom Choice logic in Blueprint:**

Derive a Blueprint subclass from `UMayDialogueChoice` and override the Event `OnChoiceSelected`. This lets you execute custom logic on every selection — without modifying the dialogue graph.

Details: [Extension → Custom Choices](../../extension/custom-choices.md).
{% endhint %}
