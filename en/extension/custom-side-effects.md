---
description: Custom SideEffects via Blueprint — using "Quest Progress Update" as an example.
---

# Custom SideEffects

SideEffects are inline actions that execute when a node is entered. For project-specific actions — quest updates, inventory changes, achievements, analytics events — you create your own subclasses.

Example goal: a SideEffect `BP_SE_QuestProgressUpdate` that increases quest progress by a configurable amount.

---

## Step 1 — Create Blueprint Class

1. Content Browser → right-click → **Blueprint Class**.
2. Open **All Classes**.
3. Search for and select `MayDialogueSideEffect`.
4. Name it: `BP_SE_QuestProgressUpdate`.

> 📸 **Image placeholder:** `cse-step1-picker.png` — "Pick Parent Class" dialog with `MayDialogueSideEffect`.
> *Setup:* Blueprint class dialog, "All Classes" tab. Search field: "MayDialogueSideEffect". Result: `UMayDialogueSideEffect` with full path. Select button active, mouse cursor on the entry.

---

## Step 2 — Add Properties

Variables panel:

* `QuestID` — type `FName`, **Instance Editable = true**, Category = "Quest"
* `ProgressDelta` — type `int32`, **Instance Editable = true**, Default value = 1, Category = "Quest"

> 📸 **Image placeholder:** `cse-step2-variables.png` — Variables panel with `QuestID` and `ProgressDelta`.
> *Setup:* Blueprint editor. Variables panel shows `QuestID` (Name) and `ProgressDelta` (Integer, Default=1). Details panel on the right: `Instance Editable = true`, `Category = Quest`. Both variables visible.

---

## Step 3 — Override ExecuteSideEffect

Functions panel → **Override** → `Execute Side Effect`.

You access the `Context` input via helper nodes from `MayDialogueLibrary` (see [Custom Requirements](custom-requirements.md)) or directly — all three Context fields have been `BlueprintReadOnly` since v1.0.

For GAS-related SideEffects (e.g. modifying attributes, firing cues), `UMayDialogueGASLibrary` is available — in particular, `Get ASC From Context` saves the manual cast to `UAbilitySystemComponent`.

Pseudo-graph:

```text
Event Execute Side Effect (Context)
  │
  ├─ Is QuestID valid? (None check)
  │     └─ false → Return (early exit)
  │
  ├─ Get World From Context  →  Get Quest Subsystem
  │
  └─ Add Quest Progress (QuestID, ProgressDelta)
```

> 📸 **Image placeholder:** `cse-step3-graph.png` — Complete Execute Side Effect graph.
> *Setup:* `Event Execute Side Effect (Context)` → `Is Valid Check (QuestID != None)` → Branch. False path: `Return`. True path: `Get World` → `Get Quest Subsystem` → `Add Quest Progress` with pinned `QuestID` and `ProgressDelta` variables. All exec pins connected, no open path.

{% hint style="info" %}
**Exit early on invalid values.** Check at the start whether all required values are set (QuestID != None, ProgressDelta != 0). A no-op is better than a crash or unexpectedly executed logic.
{% endhint %}

---

## Step 4 — Use in a Dialogue

1. Open a dialogue asset.
2. Select a node (e.g. SayLine) → Details panel → SideEffects array → **+** → `BP_SE_QuestProgressUpdate`.
3. Enter `QuestID`, set `ProgressDelta`.
4. Compile and test.

> 📸 **Image placeholder:** `cse-step4-in-graph.png` — SayLine with QuestProgressUpdate pill in the SideEffects array.
> *Setup:* MayDialogue editor, SayLine "The dragon has fallen." selected. SideEffects array expanded, one entry: `BP_SE_QuestProgressUpdate`. Below: `QuestID = KillDragon`, `ProgressDelta = 1`. Pill in the node graph shows "QuestProgress: KillDragon +1".

---

## Combining Multiple SideEffects

You can stack any number of SideEffects on a single node — they run in array order:

```text
SayLine "You have proven yourself."
  SideEffects:
    1. Quest Progress Update   QuestID=ProveYourself   ProgressDelta=1
    2. Grant Achievement       AchievementID=Proven
    3. Add Gameplay Tag        TagToAdd=Story.Proven    bAddToInstigator=true
    4. Trigger Gameplay Cue   CueTag=GameplayCue.UI.Achievement
```

> 📸 **Image placeholder:** `cse-stacked-sideeffects.png` — SayLine with four stacked SideEffects in the Details panel.
> *Setup:* MayDialogue editor, SayLine "You have proven yourself." selected. SideEffects array shows four numbered entries: 1. `QuestProgressUpdate (ProveYourself, +1)`, 2. `GrantAchievement (Proven)`, 3. `AddTag (Story.Proven)`, 4. `TriggerCue (GameplayCue.UI.Achievement)`. All fields filled in, array index visible.

---

## Server/Client Split for Multiplayer

`UMayDialogueSideEffect` has two overridable methods:

| Method | When Called |
| --- | --- |
| `Execute Side Effect` | On the server (gameplay logic) |
| `Execute Client Side Effect` | On every client (visual effects, sounds) |

For single-player projects, both always run. For multiplayer: quest progress, inventory changes → `Execute Side Effect`. Sounds, particles → `Execute Client Side Effect`.

---

## Typical Project-Specific SideEffects

| SideEffect | Action |
| --- | --- |
| `SE_QuestProgressUpdate(QuestID, Delta)` | Increase quest progress |
| `SE_GrantAchievement(AchievementID)` | Unlock achievement |
| `SE_Inventory_Add(ItemID, Count)` | Add item |
| `SE_Inventory_Remove(ItemID, Count)` | Remove item |
| `SE_NPCReputation(FactionTag, Delta)` | Modify reputation value |
| `SE_UnlockDoor(DoorTag)` | Open door |
| `SE_NotifyAnalytics(EventName)` | Send analytics event |

Each of these is a small, focused Blueprint class.

---

## C++ Variant

Setup fundamentals (Build.cs, BlueprintNativeEvent pattern, module reload) → [C++ Extension — Fundamentals](cpp-fundamentals.md).

### Header (`MySE_QuestProgressUpdate.h`)

```cpp
#pragma once

#include "CoreMinimal.h"
#include "SubNodes/MayDialogueSideEffect.h"
#include "MySE_QuestProgressUpdate.generated.h"

struct FMayDialogueContext;

UCLASS(BlueprintType, EditInlineNew, meta=(DisplayName="Quest Progress Update"))
class MYGAME_API UMySE_QuestProgressUpdate : public UMayDialogueSideEffect
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Quest")
    FName QuestID;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Quest", meta=(ClampMin="-1000", ClampMax="1000"))
    int32 ProgressDelta = 1;

    // Server-side: gameplay mutation
    virtual void ExecuteSideEffect_Implementation(const FMayDialogueContext& Context) override;

    // Client-side: purely cosmetic (sound, particles, UI toast)
    virtual void ExecuteClientSideEffect_Implementation(const FMayDialogueContext& Context) override;

    virtual FText GetDisplayDescription_Implementation() const override;
};
```

### Implementation (`MySE_QuestProgressUpdate.cpp`)

```cpp
#include "MySE_QuestProgressUpdate.h"
#include "Instance/MayDialogueInstance.h"
#include "Types/MayDialogueTypes.h"
#include "MyGame/Quest/QuestSubsystem.h"
#include "Kismet/GameplayStatics.h"

void UMySE_QuestProgressUpdate::ExecuteSideEffect_Implementation(const FMayDialogueContext& Context)
{
    if (QuestID == NAME_None || ProgressDelta == 0) return;

    UWorld* World = Context.DialogueInstance ? Context.DialogueInstance->GetWorld() : nullptr;
    if (UQuestSubsystem* Quest = World ? UQuestSubsystem::Get(World) : nullptr)
    {
        Quest->AddProgress(QuestID, ProgressDelta);
    }
}

void UMySE_QuestProgressUpdate::ExecuteClientSideEffect_Implementation(const FMayDialogueContext& Context)
{
    if (QuestID == NAME_None) return;

    // Example: play a quest progress toast sound on every client.
    if (UWorld* World = Context.DialogueInstance ? Context.DialogueInstance->GetWorld() : nullptr)
    {
        UGameplayStatics::PlaySound2D(World, /*USoundBase* */ nullptr /* your cue here */);
    }
}

FText UMySE_QuestProgressUpdate::GetDisplayDescription_Implementation() const
{
    return FText::Format(
        NSLOCTEXT("MyGame", "QuestProgressDesc", "Quest Progress: {0} {1}{2}"),
        FText::FromName(QuestID),
        ProgressDelta >= 0 ? FText::FromString(TEXT("+")) : FText::GetEmpty(),
        FText::AsNumber(ProgressDelta));
}
```

{% hint style="info" %}
**`BlueprintNativeEvent` — no UFUNCTION override in the subclass.** Only override the `_Implementation` symbols. Both methods (`ExecuteSideEffect_Implementation` AND `ExecuteClientSideEffect_Implementation`) can be overridden separately or together — the non-overridden one falls back to the base class default (no-op).
{% endhint %}

### Server/Client Call Behavior

`UMayDialogueNode_Base::ExecuteNode_Implementation` calls SideEffects in this order:

1. **Server-Authoritative Path:** `ExecuteSideEffect_Implementation` runs on the server (or Standalone). Replication, GAS mutations, and SaveGame writes belong here.
2. **Client-Replicated Path:** `ExecuteClientSideEffect_Implementation` runs on every client (including the listen server host) as a cosmetic multicast. Sounds, particles, and UI toasts belong here.

In a single-player build, both run on the same machine — no difference. In multiplayer you will be glad the separation is already in the API.

### Mixing C++ and Blueprint

Same pattern as for Requirements: C++ base (e.g. `UMySE_QuestActionBase` with shared Quest Subsystem resolution) + Blueprint subclasses (`BP_SE_QuestProgress`, `BP_SE_QuestComplete`, `BP_SE_QuestFail`). Designers iterate in Blueprint, the expensive plumbing is implemented once in C++.

---

## Executing Multiple SideEffects in a Custom Node Workflow

When you need to execute all SideEffects of a node collection correctly in a custom workflow (e.g. an async node or an external manager script), use `UMayDialogueAsyncLibrary`:

| Function | When |
| --- | --- |
| `Execute All Side Effects (Context, SideEffects)` | Server-side execution of all SideEffects in an array |
| `Execute All Client Side Effects (Context, SideEffects)` | Client-side (cosmetic) execution |

These wrappers respect the server/client separation and are directly Blueprint-callable. Normally you call `Execute Side Effects (Context)` on `Self` (the node) — these library helpers are for advanced cases, e.g. when combining SideEffect arrays from multiple sources.

---

## Best Practices

* **Aim for idempotency.** If the same dialogue plays twice, your SideEffect should not break. A counter increment is inherently not idempotent — that's fine, but keep it in mind during design.
* **No user input in SideEffects.** They run inline and synchronously. No waiting, no choice presentation, no async calls.
* **Small, focused classes.** Prefer `SE_QuestProgress` and `SE_GrantAchievement` over one mega-SideEffect with ten flags.
