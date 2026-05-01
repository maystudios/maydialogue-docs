---
description: HasTag, CheckAttribute, HasAbility — three predefined GAS Requirements for dialogue conditions.
---

# GAS Requirements

Requirements decide whether a Choice, Branch, or SayLine node is active. These three classes cover the most common GAS checks — without writing a single line of code.

All Requirements return one of three states:

| Result | Meaning |
| --- | --- |
| `Passed` | Condition met — node is active. |
| `FailedButVisible` | Not met, but visible (e.g. greyed out with tooltip). |
| `FailedAndHidden` | Not met and completely invisible. |

---

## Has Gameplay Tag

**Class:** `UMayDlgRequirement_HasTag`

Checks whether an actor carries a specific GameplayTag. First via the AbilitySystemComponent, with a fallback to `IGameplayTagAssetInterface`.

### What It's For

Classic use case: a Choice appears only when the player has acquired a story tag — e.g. because they found a codex fragment or previously spoke with an informant.

### Properties

| Property | Type | Description |
| --- | --- | --- |
| `RequiredTag` | `FGameplayTag` | The tag to check. |
| `bCheckOnInstigator` | bool | `true` = player ASC; `false` = NPC ASC. |
| `bHideOnFail` | bool | `true` = FailedAndHidden; `false` = FailedButVisible. |

> 📸 **Image placeholder:** `req-hastag-details.png` — Details panel of the HasTag Requirement.
> *Setup:* Select the HasTag sub-node on a Choice. Details panel on the right shows: `RequiredTag = Story.Witness.Murder`, `bCheckOnInstigator = true` (checked), `bHideOnFail = true` (checked). Fields clearly labeled, no other sub-node visible.

### Example: Witness Choice

```text
Choice "I was there when it happened"
  Requirement: Has Gameplay Tag
    RequiredTag:        Story.Witness.Murder
    bCheckOnInstigator: true
    bHideOnFail:        true
```

The Choice only appears when the player ASC carries the tag `Story.Witness.Murder`.

### Fail Mode Comparison

| Mode | When to Use |
| --- | --- |
| `bHideOnFail = true` | Classic RPG: option only exists once the prerequisite is met. |
| `bHideOnFail = false` | Skill system: option always visible, makes progression tangible. |

---

## Check GAS Attribute

**Class:** `UMayDlgRequirement_CheckAttribute`

Compares a GAS attribute against a threshold. Supports all common comparison operators and can read BaseValue or CurrentValue.

### What It's For

Checks like "player has enough Stamina to fight" or "NPC has less than 30% Health". Particularly useful for Choices that should be visible but disabled, to show progression.

### Properties

| Property | Type | Description |
| --- | --- | --- |
| `Attribute` | `FGameplayAttribute` | The attribute to check (e.g. `UVHSAttributeSet::GetStaminaAttribute()`). |
| `ComparisonValue` | float | Threshold. |
| `ComparisonOp` | `EMayDlgComparisonOp` | `>` / `<` / `==` / `>=` / `<=` / `!=` |
| `bCheckOnInstigator` | bool | `true` = player; `false` = NPC. |
| `bUseBaseValue` | bool | `true` = BaseValue (before modifiers); `false` = CurrentValue (after modifiers). |
| `Tolerance` | float | Tolerance for `==` / `!=` comparisons (prevents floating point issues). |
| `FailResult` | `EMayDialogueRequirementResult` | What is returned on failure. |

> 📸 **Image placeholder:** `req-checkattribute-details.png` — Details panel of the CheckAttribute Requirement with filled values.
> *Setup:* Select CheckAttribute sub-node on a Choice. Details panel shows: `Attribute = Stamina (UVHSAttributeSet)`, `ComparisonOp = GreaterOrEqual (>=)`, `ComparisonValue = 30.0`, `bCheckOnInstigator = true`, `bUseBaseValue = false`, `Tolerance = 0.0001`, `FailResult = FailedButVisible`.

### Example: Stamina Check

```text
Choice "I can still fight"
  Requirement: Check GAS Attribute
    Attribute:          Stamina (from your AttributeSet)
    ComparisonOp:       >= (GreaterOrEqual)
    ComparisonValue:    30.0
    bCheckOnInstigator: true
    FailResult:         FailedButVisible
```

The Choice is visible but not selectable when Stamina is <= 30. An optional tooltip explains to the player why.

{% hint style="info" %}
**BaseValue vs. CurrentValue:** Use `bUseBaseValue = true` when you want to check the "permanent" value — regardless of whether a buff or debuff is currently active. For "how it feels right now" use `bUseBaseValue = false` (default).
{% endhint %}

---

## Has Gameplay Ability

**Class:** `UMayDlgRequirement_HasAbility`

Checks whether a GameplayAbility is granted on the ASC. Can match by class or by tag container.

### What It's For

Show Choices only when the player has a specific ability ("I cast a fireball" → only if `GA_Fireball` is granted). Or check whether an ability is currently active.

### Properties

| Property | Type | Description |
| --- | --- | --- |
| `RequiredAbility` | `TSubclassOf<UGameplayAbility>` | Class-based match (fallback when tag container is empty). |
| `AbilityTagsAny` | `FGameplayTagContainer` | Matches if an ability has **at least one** of these tags. |
| `AbilityTagsAll` | `FGameplayTagContainer` | Matches if an ability has **all** of these tags. |
| `bRequireActive` | bool | `true` = only abilities currently running (`IsActive()`) count. |
| `bCheckOnInstigator` | bool | `true` = player; `false` = NPC. |
| `bHideOnFail` | bool | Visibility mode on failure. |

**Match priority:** `AbilityTagsAny` → `AbilityTagsAll` → `RequiredAbility`. The class check is only active when both tag containers are empty.

> 📸 **Image placeholder:** `req-hasability-details.png` — Details panel of the HasAbility Requirement with a tag container entry.
> *Setup:* Select HasAbility sub-node on a Choice. Details panel shows: `RequiredAbility = empty`, `AbilityTagsAny = (Ability.Magic.Fire)`, `AbilityTagsAll = empty`, `bRequireActive = false`, `bCheckOnInstigator = true`, `bHideOnFail = true`.

### Example: Magic Choice

```text
Choice "I cast a fireball"
  Requirement: Has Gameplay Ability
    AbilityTagsAny:     Ability.Magic.Fire
    bCheckOnInstigator: true
    bHideOnFail:        true
```

Appears only when the player ASC has an ability with the tag `Ability.Magic.Fire`.

---

## Combining Multiple Requirements

You can stack any number of Requirements on a Choice, Branch, or SayLine. The system merges them automatically:

* All Passed → Passed.
* At least one `FailedAndHidden` → FailedAndHidden (dominates).
* Otherwise at least one `FailedButVisible` → FailedButVisible.

```text
Choice "I am the Chosen One"
  Requirements:
    1. Has Gameplay Tag:     Story.Chosen.Marked        bHideOnFail: true
    2. Check GAS Attribute:  Reputation >= 80           FailResult: FailedAndHidden
```

If either is missing: Choice is invisible. If both are met: Choice appears.

{% hint style="info" %}
**Want to build custom Requirements?** Create a Blueprint class with parent `UMayDialogueRequirement` and override `Is Requirement Satisfied`. More in [Creating Custom GAS Nodes](extending.md).
{% endhint %}
