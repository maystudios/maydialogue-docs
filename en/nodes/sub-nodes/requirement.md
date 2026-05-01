# Requirement

A Requirement is a condition check that can deliver three results: passed, failed but visible, or completely hidden. You place Requirements on Nodes and Choices to control the dialogue flow and the availability of options.

## When should I use it?

- As a Condition on a [Branch Node](../core/branch.md) to automatically choose between paths.
- On a [Choice](choice.md) to make an option visible only for players with the right tag or attribute.
- On any Node to skip it if the player does not have a certain state.
- Multiple Requirements on the same Node/Choice are combined: the strictest result wins.

## The Three Results

| Result | Meaning |
| --- | --- |
| `Passed` | Condition met — Node/Choice is executed/displayed normally. |
| `FailedButVisible` | Not met — Choice is displayed greyed out (with `UnavailableReason` tooltip). Node is skipped or aborted (depending on `FailBehavior`). |
| `FailedAndHidden` | Not met — Choice is not displayed at all. Node is skipped. |

## Properties (Base Class)

| Property | Type | Default | Meaning |
| --- | --- | --- | --- |
| `Description` | `FText` | empty | Editor tooltip / pill label in the graph. |
| `FailResult` | `EMayDialogueRequirementFailResult` | `FailedButVisible` | What happens on failure: `FailedButVisible` or `FailedAndHidden`. |

{% hint style="warning" %}
`bHideOnFail` is deprecated — use the `EMayDialogueRequirementResult` field `FailResult = FailedAndHidden` instead.
{% endhint %}

## Built-in Requirements (GAS Integration)

These Requirements are available out of the box — no additional setup needed. They access the Gameplay Ability System if your project uses it.

| Class | Checks | Key Properties |
| --- | --- | --- |
| `UMayDlgRequirement_HasTag` | ASC carries a specific GameplayTag | `RequiredTag`, `bCheckOnInstigator` |
| `UMayDlgRequirement_CheckAttribute` | Attribute comparison (>, <, ==, >=, <=, !=) | `Attribute`, `ComparisonOp`, `ComparisonValue`, `bCheckOnInstigator` |
| `UMayDlgRequirement_HasAbility` | ASC has a specific ability class | `RequiredAbility`, `bCheckOnInstigator` |

Details: [GAS → Requirements](../../gas/requirements.md).

> 📸 **Image placeholder:** `requirement-node-pill.png` — Requirement as a pill on a Branch Node.
> *Setup:* Select Branch Node in the graph. Visible in Node body: Condition pill `HasTag Story.Met.Guard`. Details panel on the right shows: `RequiredTag = Story.Met.Guard`, `FailResult = FailedButVisible`, `Description = "Has the player already met the guard?"`.

> 📸 **Image placeholder:** `requirement-details-panel.png` — Details panel of a HasTag Requirement.
> *Setup:* Expand the Requirement in the Branch Node's Details panel. Visible: `Description`, `FailResult = FailedAndHidden`, `RequiredTag = Story.HeardPassword`, `bCheckOnInstigator = true`.

> 📸 **Image placeholder:** `requirement-on-choice-pill.png` — Choice pill with attached Requirement (as pill in Choice body).
> *Setup:* PlayerChoice Node. Choice pill `"Password: 'Shadow Door'."` expanded. In the Choice body: Requirement pill `HasTag Story.HeardPassword (FailedAndHidden)`. Shows the parent Node (PlayerChoice) with attached Sub-Node (Requirement) in the pill.

## Mini Example

**On Branch (automatic switch):**

```text
[Branch]
  Condition: HasTag "Story.Met.Guard"
  ├─ True  ──► [SayLine: "Good to see you again."]
  └─ False ──► [SayLine: "Halt! Who are you?"]
```

**On Choice (availability):**

```text
[PlayerChoice]
  ├─ Choice 0 "Friend"                       (no Requirements)
  ├─ Choice 1 "Give password"                (HasTag Story.HeardPassword, FailedAndHidden)
  └─ Choice 2 "Persuade with reputation"     (CheckAttribute Reputation >= 50, FailedButVisible)
      UnavailableReason: "Your reputation is not high enough."
```

> 📸 **Image placeholder:** `requirement-example-graph.png` — PlayerChoice with three Choices and different Requirements.
> *Setup:* PlayerChoice Node, three Choices expanded. Choice 0: no pill. Choice 1: Requirement pill `HasTag Story.HeardPassword`. Choice 2: Requirement pill `CheckAttribute Reputation >= 50`. `UnavailableReason` for Choice 2 visible in the Details panel.

## Combining Multiple Requirements

Requirements on the same Node/Choice are combined with `EvaluateAll`:

- All `Passed` → `Passed`.
- Any `FailedAndHidden` → `FailedAndHidden` (strongest result wins).
- Otherwise any `FailedButVisible` → `FailedButVisible`.

## Common Pitfalls

- **`FailResult` not set**: Default is `FailedButVisible` — the Choice/Node is visible but disabled. If you want to hide it entirely, set `FailResult = FailedAndHidden`.
- **`bCheckOnInstigator` forgotten**: Without this flag, the Requirement checks on the dialogue's Target (e.g. NPC), not the player. Set `bCheckOnInstigator = true` for player-related checks.

## Extending

{% hint style="success" %}
**Custom Requirement logic in Blueprint:**

1. Right-click in Content Browser → Blueprint Class → Parent: `UMayDialogueRequirement`.
2. Override Event `IsRequirementSatisfied` — returns `Passed`, `FailedButVisible`, or `FailedAndHidden`.
3. Compile Blueprint.
4. In the Details panel of a Node/Choice → "Add Requirement" → your new class appears in the list.

Details: [Extension → Custom Requirements](../../extension/custom-requirements.md).
{% endhint %}

```cpp
// C++ override (Advanced):
virtual EMayDialogueRequirementResult IsRequirementSatisfied_Implementation(
    const FMayDialogueContext& Context) const override;
```
