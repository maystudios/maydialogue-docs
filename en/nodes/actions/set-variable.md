---
description: Set a dialogue variable — Bool, Int, Float, String, or Tag.
---

# Set Variable

Writes a value to a named dialogue variable. The variable can be stored in dialogue scope (this instance only) or participant scope (the NPC's persistent memory).

## When to use

- **Set a secret flag** — Player hears something important: `HasHeardSecret = true` for later Branch decisions.
- **Count quest progress** — Player delivers items: increment `QuestItemCount` by 1 (Int variable).
- **Remember Speaker state** — Which answer did the player last give? (`LastChoice = "Option_A"`).
- **Tag variable for complex conditions** — Set a GameplayTag value that a later Branch Node evaluates.

---

> 📸 **Image placeholder:** `set-variable-node.png` — "Set Variable" Node in the MayDialogue graph.
> *Setup:* Node alone visible, title bar "Set Variable" (category color: yellow/Data). Subtitle shows: `HasHeardSecret = true (Bool)`. Input and Output pins visible.

---

## Properties

| Property | Type | Description |
|---|---|---|
| `VariableName` | `FName` | Name of the variable. Must be declared in the asset's Variables panel. |
| `VariableType` | `EMayDialogueVariableType` | `Bool` / `Int` / `Float` / `String` / `Tag` — determines which value field is active. |
| `Scope` | `EMayDialogueVariableScope` | `Dialogue` = this instance only. `Participant` = NPC's persistent memory. |
| `TargetParticipantTag` | `FGameplayTag` | Which Participant receives the value. Only active when `Scope = Participant`. |
| `BoolValue` | `bool` | Value when `VariableType = Bool`. |
| `IntValue` | `int32` | Value when `VariableType = Int`. |
| `FloatValue` | `float` | Value when `VariableType = Float`. |
| `StringValue` | `FString` | Value when `VariableType = String`. |
| `TagValue` | `FGameplayTag` | Value when `VariableType = Tag`. |

> The Details panel always shows only the value field matching the `VariableType` (EditConditionHides).

---

> 📸 **Image placeholder:** `set-variable-details.png` — Details panel with Bool setup.
> *Setup:* Select the Node. In the Details panel: `VariableName = HasHeardSecret`, `VariableType = Bool`, `Scope = Dialogue`, `TargetParticipantTag = (empty, greyed out)`, `BoolValue = true`. All other value fields hidden.

---

## Action Node or SideEffect Sub-Node?

If setting the variable is **the content main point** of this step (the graph step represents the moment of remembering), use the Action Node. If the variable is just set incidentally when entering a SayLine (automatic tracking), attach a SideEffect Sub-Node to the SayLine.

---

## Example: Setting a secret flag

```text
[SayLine: Old Man "Listen carefully..."]
  │
  ▼
[SetVariable: HasHeardSecret=true, Scope=Dialogue]
  │
  ▼
[SayLine: Old Man "The key is in the tower."]
  │
  ▼
[Branch: Condition=HasHeardSecret → True → [Exit Completed]]
```

> 📸 **Image placeholder:** `set-variable-example-graph.png` — Graph snippet of the example above.
> *Setup:* Four Nodes: SayLine → SetVariable (HasHeardSecret=true in subtitle) → SayLine → Branch. All pins connected. Branch Node shows True/False output pins.

---

## Pitfalls

{% hint style="warning" %}
`VariableName` must be declared in the **Variables panel** of the dialogue asset. An unknown name results in a log warning and the variable is not written — no hard error, but silently corrupted logic.
{% endhint %}

{% hint style="info" %}
**Participant scope** is intended for persistent NPC memory (NPC remembers after the dialogue ends). Writing to this scope only works if the Participant has a corresponding PersistentMemory component.
{% endhint %}

- When changing `VariableType`, the other value fields are hidden — the respective value remains stored in the asset but is not used.
- `OnVariableChanged` is broadcast after writing — external systems can react to it.
