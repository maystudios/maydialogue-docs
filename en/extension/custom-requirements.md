---
description: Custom Requirements via Blueprint — using "Quest Completed" as an example.
---

# Custom Requirements

Requirements control whether a Choice, Branch, or SayLine is active. You can create custom Requirements for any project-specific condition: quest status, inventory, time of day, reputation value.

Example goal: a Choice appears only when the player has completed a specific quest — `BP_Req_QuestCompleted`.

---

## Step 1 — Create Blueprint Class

1. Content Browser → right-click → **Blueprint Class**.
2. Open **All Classes**.
3. Search for and select `MayDialogueRequirement`.
4. Name it: `BP_Req_QuestCompleted`.

> 📸 **Image placeholder:** `creq-step1-picker.png` — "Pick Parent Class" dialog with `MayDialogueRequirement`.
> *Setup:* Blueprint class dialog, "All Classes" tab. Search field: "MayDialogueRequirement". Result: `UMayDialogueRequirement` with module path visible. Select button active.

---

## Step 2 — Add Properties

Variables panel → new variable:

* `QuestID` — type `FName`, **Instance Editable = true**, Category = "Quest"
* Optional: `RequiredStage` — type `int32`, Instance Editable = true, Default value = 0, Category = "Quest"

> 📸 **Image placeholder:** `creq-step2-variables.png` — Variables panel with `QuestID` and `RequiredStage`.
> *Setup:* Blueprint editor. Variables panel shows `QuestID` (Name, highlighted) and `RequiredStage` (Integer, default value 0). Details panel on the right: `Instance Editable = true`, `Category = Quest`.

---

## Step 3 — Override IsRequirementSatisfied

Functions panel → **Override** → `Is Requirement Satisfied`.

You access the `Context` input in the Blueprint graph either directly (all three fields — `Dialogue Instance`, `Instigator`, `Target` — have been `BlueprintReadOnly` since v1.0) or via helper nodes from `MayDialogueLibrary`:

| Helper Node (BP) | What It Returns |
| --- | --- |
| `Get Dialogue Instance (Context)` | `UMayDialogueInstance*` |
| `Get World From Context` | `UWorld*` (equivalent to Context.GetWorld()) |
| `Get Dialogue Asset (Context)` | `UMayDialogueAsset*` (Context → Instance → Asset) |
| `Resolve Instigator ASC (Context)` | `UObject*` — cast to `AbilitySystemComponent` on the Blueprint side |
| `Get Dialogue Variable (Bool/Int/Float/String/Tag)` | direct variable read with default fallback |

For GAS-related Requirements, `UMayDialogueGASLibrary` (category `MayDialogue|GAS`) is available:

| GAS Helper (BP) | What It Returns |
| --- | --- |
| `Get ASC From Context` | `UAbilitySystemComponent*` directly from the Context — no manual cast needed |
| `Is Dialogue Server Authoritative` | `bool` — whether the current context is server-authoritative (multiplayer branching) |
| `Trigger Cue On Context` | Fire a Gameplay Cue directly from a Requirement override |

Pseudo-graph:

```text
Event Is Requirement Satisfied (Context)
  │
  ├─ Is QuestID valid? (None check)
  │     └─ true → Return Passed  (no quest = always passed)
  │
  ├─ Get World From Context  →  Get Quest Subsystem
  │
  ├─ Is Quest Completed? (QuestID, RequiredStage)
  │     ├─ true  → Return Passed
  │     └─ false → Get Fail Result (Self)  →  Return  (Hidden vs Visible from base class)
```

> 📸 **Image placeholder:** `creq-step3-graph.png` — Complete Is Requirement Satisfied graph.
> *Setup:* `Event Is Requirement Satisfied (Context)` → `Is Valid QuestID` (None comparison) → Branch. True path directly: `Return Passed`. False path: `Get World → Get Quest Subsystem → Is Quest Completed (QuestID)` → Branch. True: `Return Passed`. False: `Branch: bHideOnFail` → two Return nodes (`FailedAndHidden` / `FailedButVisible`). All paths connected, no open exec pins.

{% hint style="info" %}
`bHideOnFail` is included in the base class `UMayDialogueRequirement` and does not need to be declared yourself. It appears in the Details panel and controls whether `FailedAndHidden` or `FailedButVisible` is returned.
{% endhint %}

---

## Step 4 — Use in a Dialogue

1. Open a dialogue asset.
2. Select a Choice → Details panel → **Add Requirement** → `BP_Req_QuestCompleted`.
3. Enter `QuestID` (e.g. `KillDragon`).
4. Set `bHideOnFail` to true or false depending on your design.
5. Compile and test in the Preview Runner.

> 📸 **Image placeholder:** `creq-step4-details.png` — Details panel of a Choice with `BP_Req_QuestCompleted` as a sub-node.
> *Setup:* MayDialogue editor, Choice "I killed the dragon" selected. Requirements array in the Details panel shows one entry: `BP_Req_QuestCompleted`. Below: `QuestID = KillDragon`, `RequiredStage = 0`, `bHideOnFail = true`. Fields clearly readable, pill label in the graph shows "QuestCompleted: KillDragon".

---

## Composable Requirements — Stacking Multiple

Multiple Requirements can be placed on the same Choice:

```text
Choice "I have prepared everything"
  Requirements:
    1. Quest Completed: KillDragon          bHideOnFail: true
    2. Quest Completed: CollectIngredients  bHideOnFail: true
    3. Check GAS Attribute: Gold >= 100     FailResult: FailedButVisible
```

Result: the Choice only appears when both quests are completed. If Gold is too low: Choice visible but disabled (with tooltip).

> 📸 **Image placeholder:** `creq-stacked-requirements.png` — Details panel of a Choice with three stacked Requirements.
> *Setup:* Choice node in the MayDialogue editor selected. Requirements array shows three entries: `BP_Req_QuestCompleted (KillDragon, bHideOnFail=true)`, `BP_Req_QuestCompleted (CollectIngredients, bHideOnFail=true)`, `UMayDlgRequirement_CheckAttribute (Gold >= 100, FailResult=FailedButVisible)`. All three visible, array elements numbered.

---

## Typical Project-Specific Requirements

| Requirement | Checks |
| --- | --- |
| `Req_QuestActive(QuestID)` | Quest is currently active |
| `Req_QuestCompleted(QuestID)` | Quest completed (example above) |
| `Req_InventoryContains(ItemID, Count)` | Player has item |
| `Req_TimeOfDay(MinHour, MaxHour)` | Time of day within range |
| `Req_AreaVisited(AreaTag)` | Player has visited this area |
| `Req_NPCReputation(NPCTag, MinLevel)` | Reputation value high enough |

Each of these is a small Blueprint class that designers select directly from the dropdown menu.

---

## C++ Variant

For complex or performance-critical Requirements, C++ is the better path. Setup fundamentals (Build.cs, BlueprintNativeEvent pattern, module reload) are in [C++ Extension — Fundamentals](cpp-fundamentals.md).

### Header (`MyReq_QuestCompleted.h`)

```cpp
#pragma once

#include "CoreMinimal.h"
#include "SubNodes/MayDialogueRequirement.h"
#include "MyReq_QuestCompleted.generated.h"

struct FMayDialogueContext;

UCLASS(BlueprintType, EditInlineNew, meta=(DisplayName="Quest Completed"))
class MYGAME_API UMyReq_QuestCompleted : public UMayDialogueRequirement
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Quest")
    FName QuestID;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Quest")
    int32 RequiredStage = 0;

    virtual EMayDialogueRequirementResult IsRequirementSatisfied_Implementation(
        const FMayDialogueContext& Context) const override;

    virtual FText GetDisplayDescription_Implementation() const override;
};
```

### Implementation (`MyReq_QuestCompleted.cpp`)

```cpp
#include "MyReq_QuestCompleted.h"
#include "Instance/MayDialogueInstance.h"
#include "Types/MayDialogueTypes.h"
#include "MyGame/Quest/QuestSubsystem.h"

EMayDialogueRequirementResult UMyReq_QuestCompleted::IsRequirementSatisfied_Implementation(
    const FMayDialogueContext& Context) const
{
    if (QuestID == NAME_None)
    {
        return EMayDialogueRequirementResult::Passed;
    }

    UWorld* World = Context.DialogueInstance ? Context.DialogueInstance->GetWorld() : nullptr;
    UQuestSubsystem* Quest = World ? UQuestSubsystem::Get(World) : nullptr;
    if (!Quest)
    {
        return EMayDialogueRequirementResult::Passed;
    }

    if (Quest->IsCompletedAtStage(QuestID, RequiredStage))
    {
        return EMayDialogueRequirementResult::Passed;
    }

    // GetFailResult() resolves Hidden vs Visible from the FailResult property of the base class
    // and respects the deprecated bHideOnFail flag for migration.
    return GetFailResult();
}

FText UMyReq_QuestCompleted::GetDisplayDescription_Implementation() const
{
    return FText::Format(
        NSLOCTEXT("MyGame", "QuestCompletedDesc", "Quest completed: {0} (Stage >= {1})"),
        FText::FromName(QuestID),
        FText::AsNumber(RequiredStage));
}
```

{% hint style="info" %}
**`BlueprintNativeEvent` — no UFUNCTION override.** You only override the `_Implementation` symbol, NOT the bare `IsRequirementSatisfied`. If you mark it with `UFUNCTION(...)` anyway, UHT gives a "redundant override" error. See [cpp-fundamentals §2](cpp-fundamentals.md#2-blueprintnativeevent-what-how-why).
{% endhint %}

`GetDisplayDescription` returns the text that appears as a pill label in the graph. Populating it dynamically with the current property values (`"Quest completed: KillDragon (Stage >= 2)"`) is a hundred times more helpful than the static class name.

### Mixing C++ and Blueprint

You can create a C++ base with `Blueprintable` and `Abstract`, then derive concrete Blueprint subclasses from it. Example: `UMyReq_QuestBase` (C++, abstract, with shared subsystem resolution) → `BP_Req_QuestActive`, `BP_Req_QuestCompleted`, `BP_Req_QuestStageReached` (Blueprints, each with different `IsRequirementSatisfied` logic). Designers work in Blueprint, the expensive subsystem lookup is implemented once in C++.
