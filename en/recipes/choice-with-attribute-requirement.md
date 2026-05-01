---
description: Make a PlayerChoice option selectable only when a GAS attribute reaches a threshold.
---

# Choice with Attribute Condition

## Scenario

An alchemist offers the player a powerful potion. The purchase option should always be visible — but only selectable when the player has at least 50 Stamina. If Stamina is below 50, the player sees the choice grayed out with the hint *"You are too exhausted."* This is the classic RPG pattern: the player knows what they haven't achieved yet.

## What You Will Learn

- Configure a CheckAttribute Requirement on a Choice.
- Combine `FailedButVisible` with `UnavailableReason`.
- The difference from `FailedAndHidden` (from the Choice Tag Requirement recipe).
- An overview of attribute comparison operators.

## Prerequisites

- [Branching with Conditions](branching-conditions.md) completed.
- GAS active, AttributeSet with `Stamina` registered and attached to the player's ASC.

## Mini-Graph

```text
[Entry]
   │
   ▼
[SayLine: "What are you seeking, wanderer?"]
   │
   ▼
[PlayerChoice]
   ├─ "The ordinary potion."   (no Req) → [Exit: Completed]
   └─ "The powerful potion."   (Req: Stamina >= 50, FailedButVisible)
         └─► [SayLine: "Very well. 30 gold."] → [Exit: Completed]
```

> 📸 **Image placeholder:** `choice-with-attribute-requirement-graph-overview.png` — Graph with PlayerChoice and an attribute-locked choice.
> *Setup:* Asset `DA_Alchemist_Shop` open. SayLine → PlayerChoice with two choices. Second choice has lock icon (yellow = FailedButVisible). Details panel: CheckAttribute Requirement visible.

## Step by Step

### 1. Create the Asset and Speaker

Asset: `DA_Alchemist_Shop`. Speaker: `Dialogue.Speaker.Alchemist`.

### 2. PlayerChoice with Two Choices

SayLine *"What are you seeking, wanderer?"* → PlayerChoice. Two choices via **Add Choice**.

Choice 1: `ChoiceText = "The ordinary potion."` — no Requirement.

Choice 2: `ChoiceText = "The powerful potion."` — with Requirement (next step).

### 3. Configure the CheckAttribute Requirement

On Choice 2 under **Requirements → Add → CheckAttribute**:

| Property | Value |
|----------|------|
| `Attribute` | `UVHSAttributeSet::Stamina` (dropdown) |
| `ComparisonOp` | `>=` |
| `ComparisonValue` | `50.0` |
| `FailureResult` | `FailedButVisible` |
| `UnavailableReason` | *"You are too exhausted."* |
| `bCheckOnInstigator` | `true` |

> 📸 **Image placeholder:** `choice-with-attribute-requirement-details.png` — Details panel of the second choice with CheckAttribute Requirement.
> *Setup:* PlayerChoice selected, Choice 2 expanded. `ChoiceText = "The powerful potion."`. Requirements: `CheckAttribute: Attribute = Stamina, ComparisonOp = >=, ComparisonValue = 50.0, FailureResult = FailedButVisible, UnavailableReason = "You are too exhausted."`.

### 4. Wire Up the Outputs

Choice 1 → Exit. Choice 2 → SayLine *"Very well. 30 gold."* → Exit.

### 5. Compile and Test

In the Preview Runner: Stamina below 50 → Choice 2 visible but grayed out, hover shows the reason text. Stamina above 50 → choice is selectable.

## Available Comparison Operators

| Operator | Meaning |
|----------|-----------|
| `==` | Exactly equal |
| `!=` | Not equal |
| `<` | Less than |
| `<=` | Less than or equal |
| `>` | Greater than |
| `>=` | Greater than or equal |

## Blueprint Triggering

Standard dialogue start:

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_Alchemist_Shop
   └─ ...
```

> 📸 **Image placeholder:** `choice-with-attribute-requirement-ingame.png` — Preview Runner with grayed-out choice.
> *Setup:* Preview Runner with `Stamina = 30`. Choice list shows both choices, second one grayed out. Hover tooltip: `"You are too exhausted."`. Set Stamina to 60 in the Debugger → choice becomes active.

{% hint style="info" %}
**C++ variant:** You can also build the Requirement directly in C++. Create a subclass of `UMayDialogueRequirement` and override `CheckRequirement()`. You can read the attribute there via `AbilitySystemComponent->GetNumericAttribute(Attribute)`.
{% endhint %}

## Variations / Going Further

- The same pattern for **gold costs**: `CheckAttribute(Currency.Gold >= 30)`, `FailedButVisible`, reason *"Not enough gold."*
- Combine multiple attributes (AND): attach a second Requirement to the same choice.
- On success, deduct Stamina: SideEffect on the choice → `ApplyEffect(GE_StaminaCost)`.

## Troubleshooting

**Choice always grayed out, even though the attribute is high enough.**
`bCheckOnInstigator = false` → attribute is checked on the NPC's ASC, not the player's. Set to `true`. Also: attribute dropdown empty or incorrectly linked — the AttributeSet must be registered on the ASC.

**Attribute dropdown doesn't show the attribute.**
The AttributeSet is not registered on the player's ASC. Check `UAbilitySystemComponent::AddAttributeSetSubobject()` or `DefaultStartingData` in GameplayAbility Settings.

**UnavailableReason is not shown in the widget.**
The dialogue widget must read `UnavailableReason` from the choice data and display it. This is implemented in the standard Slate debug widget; with a custom UMG widget you need to bind the tooltip yourself.
