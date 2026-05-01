---
description: Custom dialogue nodes via Blueprint — step by step.
---

# Custom Nodes

The predefined nodes (SayLine, PlayerChoice, Branch, …) are not sufficient for every scenario. When your graph needs to notify a quest system or start a level sequence, for example, you create a custom node type.

Example goal: a node `BP_DN_NotifyQuest` that marks a quest step as "reached" when traversed.

---

## Step 1 — Create Blueprint Class

1. Content Browser → right-click → **Blueprint Class**.
2. Open **All Classes** in the class picker.
3. Type `MayDialogueNode_Base` and select it.
4. Name it: `BP_DN_NotifyQuest`.

> 📸 **Image placeholder:** `cnode-step1-picker.png` — "Pick Parent Class" dialog with `MayDialogueNode_Base` selected in the search.
> *Setup:* Blueprint class dialog. "All Classes" tab active. Search field: "MayDialogueNode_Base". Result list shows the class with its full module path. Mouse cursor on the entry, Select button highlighted.

---

## Step 2 — Add Properties

In the Variables panel:

* `QuestStepName` — type `FName`, **Instance Editable = true**, Category = "Quest"
* Optional: `bMarkCompleted` — type `bool`, Instance Editable = true, Category = "Quest"

> 📸 **Image placeholder:** `cnode-step2-variables.png` — Variables panel with `QuestStepName` and `bMarkCompleted`.
> *Setup:* Blueprint editor open. Variables panel on the left shows `QuestStepName` (type Name, highlighted) and `bMarkCompleted` (type Bool). On the right in the Details panel: `Instance Editable = true` (checked), `Category = Quest`.

---

## Step 3 — Override ExecuteNode

In the Functions panel → **Override** → `Execute Node`.

You access the `Context` input via helper nodes from `MayDialogueLibrary` (see [Custom Requirements](custom-requirements.md)) or directly — all three Context fields have been `BlueprintReadOnly` since v1.0.

Pseudo-graph:

```text
Event Execute Node (Context)
  │
  ├─ Get World From Context  →  Get Quest Subsystem
  │
  ├─ Report Quest Step Reached (QuestStepName)
  │
  ├─ Execute Side Effects (Context)    ← important: execute sub-node SideEffects
  │
  └─ Return: [Make Advance] (Get First Valid Output)
```

`Execute Side Effects` and `Get First Valid Output` are Blueprint-callable methods of `UMayDialogueNode_Base` — call them on `Self`. For the return value, use the **Make \*** nodes from `UMayDialogueLibrary` (category `MayDialogue|Task Result`):

| Make Node | When |
| --- | --- |
| `Make Advance` | Normal transition to the next node |
| `Make Abort` | Abort the dialogue |
| `Make Wait` | Pause the dialogue (async nodes) |
| `Make Pause And Present Choices` | Open the choice screen |
| `Make Advance With Choice` | Choice + direct advance |
| `Make Return To Start` | Jump back to the beginning of the graph |
| `Make Return To Last` | Return to the last visited node |
| `Make Return To Current` | Repeat the current node |

> 📸 **Image placeholder:** `cnode-step3-graph.png` — Complete ExecuteNode graph with quest call and return.
> *Setup:* Blueprint editor, `ExecuteNode` function open. From left: `Event Execute Node (Context)` → `Get World` (from Context.DialogueInstance) → `Get Quest Subsystem` → `Report Step Reached (QuestStepName variable)` → `Execute Side Effects (Context, Self)` → `Get First Valid Output (Context, Self)` → `Return Node` with the return value. All connections visible, no open pins.

{% hint style="warning" %}
**Don't forget `Execute Side Effects`.** If your node should support SideEffect sub-nodes (and it should), you must call `ExecuteSideEffects(Context)` explicitly. The base class does not do this automatically for you.
{% endhint %}

---

## Step 4 — Set Category and Display Name

In **Class Settings** (top right in the Blueprint editor):

* **Blueprint Node Title**: `Notify Quest`
* **Category**: `MayDialogue|Actions|MyGame`

> 📸 **Image placeholder:** `cnode-step4-classsettings.png` — Class Settings panel with node title and category entry.
> *Setup:* Blueprint editor, Class Settings tab (gear icon at the top). Fields "Blueprint Display Name" and "Category" visible. Values: Name = "Notify Quest", Category = "MayDialogue|Actions|MyGame". Surrounding Class Settings fields shown greyed out / empty.

---

## Step 5 — Use in a Dialogue

1. Open a dialogue asset.
2. Right-click in the graph → context menu → category `MyGame` → select **Notify Quest**.
3. Place the node, enter `QuestStepName`.
4. Connect the input pin from the previous node, connect the output pin to the next node.
5. Compile the dialogue (Ctrl+F9), test in the Preview Runner.

> 📸 **Image placeholder:** `cnode-step5-in-graph.png` — Fully wired graph: SayLine → NotifyQuest node → Exit.
> *Setup:* MayDialogue editor, asset `DA_QuestDialogue`. From left: `SayLine "Task completed."` (dark red) → `Notify Quest` node (blue task node, QuestStepName = "KillDragon_Deliver") → `Exit Completed` node (red). All connections horizontal, node title "Notify Quest" visible.

---

## Async Nodes (Advanced)

When your node needs to wait for an external event (e.g. an animation, a timer, a network result), it returns `Make Wait` and triggers the advance later. **This now works fully in Blueprint** via `UMayDialogueAsyncLibrary`:

```text
Event Execute Node (Context)
  │
  ├─ Store NextNodeGuid = Get First Valid Output (Self, Context)
  │
  ├─ Start Timer / Bind Event
  │
  └─ Return: [Make Wait]

[When Timer Expires / Event Triggers]
  │
  └─ Request Node Advance (Context.DialogueInstance, NextNodeGuid)
       ↑ from UMayDialogueAsyncLibrary — Blueprint callable, null-guarded
```

`Request Node Advance` (category `MayDialogue|Async`) is a thin Blueprint wrapper around `Instance->ForceTransitionToNode`. It adds a null guard and is easy to find in the node palette by its discovery name.

For multiplayer projects: when you want to combine multiple SideEffects on an async node, use `Execute All Side Effects` and `Execute All Client Side Effects` from `UMayDialogueAsyncLibrary` to keep the client/server split correct.

---

## C++ Variant

Setup fundamentals (Build.cs, BlueprintNativeEvent pattern, module reload, editor visibility) → [C++ Extension — Fundamentals](cpp-fundamentals.md).

### Header (`MyDN_NotifyQuest.h`)

```cpp
#pragma once

#include "CoreMinimal.h"
#include "Nodes/MayDialogueNode_Base.h"
#include "MyDN_NotifyQuest.generated.h"

struct FMayDialogueContext;
struct FMayDialogueTaskResult;

UCLASS(BlueprintType, meta=(DisplayName="Notify Quest"))
class MYGAME_API UMyDN_NotifyQuest : public UMayDialogueNode_Base
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Quest")
    FName QuestStepName;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Quest")
    bool bMarkCompleted = false;

    virtual FMayDialogueTaskResult ExecuteNode_Implementation(
        const FMayDialogueContext& Context) override;

    virtual FText GetNodeDisplayName_Implementation() const override;
    virtual FLinearColor GetNodeColor_Implementation() const override;
};
```

### Implementation (`MyDN_NotifyQuest.cpp`)

```cpp
#include "MyDN_NotifyQuest.h"
#include "Instance/MayDialogueInstance.h"
#include "Types/MayDialogueTypes.h"
#include "MyGame/Quest/QuestSubsystem.h"

FMayDialogueTaskResult UMyDN_NotifyQuest::ExecuteNode_Implementation(
    const FMayDialogueContext& Context)
{
    if (QuestStepName != NAME_None)
    {
        UWorld* World = Context.DialogueInstance ? Context.DialogueInstance->GetWorld() : nullptr;
        if (UQuestSubsystem* Quest = World ? UQuestSubsystem::Get(World) : nullptr)
        {
            if (bMarkCompleted) Quest->MarkCompleted(QuestStepName);
            else                Quest->ReportStepReached(QuestStepName);
        }
    }

    // Execute sub-node SideEffects — the base class does NOT do this automatically
    // when you override ExecuteNode. Otherwise designer-configured SideEffects
    // on the node would be ignored.
    ExecuteSideEffects(Context);

    // FailBehavior is handled in the engine via pre-execution Requirements evaluation —
    // here you only return the success path.
    // FMayDialogueTaskResult::Advance / Abort / Wait / PauseAndPresentChoices etc.
    // are the C++ static factories. In Blueprint these are available as Make Advance / Make Abort /
    // Make Wait / Make Pause And Present Choices etc. in UMayDialogueLibrary.
    return FMayDialogueTaskResult::Advance(GetFirstValidOutput(Context));
}

FText UMyDN_NotifyQuest::GetNodeDisplayName_Implementation() const
{
    if (QuestStepName == NAME_None)
    {
        return NSLOCTEXT("MyGame", "NotifyQuestEmpty", "Notify Quest (no step)");
    }
    return FText::Format(
        NSLOCTEXT("MyGame", "NotifyQuestFmt", "Notify Quest: {0}{1}"),
        FText::FromName(QuestStepName),
        bMarkCompleted ? FText::FromString(TEXT(" ✓")) : FText::GetEmpty());
}

FLinearColor UMyDN_NotifyQuest::GetNodeColor_Implementation() const
{
    return FLinearColor(0.2f, 0.45f, 0.8f);   // Quest blue, clearly distinct from SayLine/Branch
}
```

{% hint style="info" %}
**`BlueprintNativeEvent` — no UFUNCTION override in the subclass.** Override `ExecuteNode_Implementation`, NOT `ExecuteNode`. If you mark it with `UFUNCTION(...)` anyway, UHT gives an error. `GetNodeDisplayName` and `GetNodeColor` follow the same pattern.
{% endhint %}

{% hint style="warning" %}
**Don't forget `ExecuteSideEffects(Context)`.** When you override `ExecuteNode_Implementation`, you take full control — the base class no longer calls the SideEffect sub-nodes for you. Do NOT forget this call, otherwise designer-configured SideEffects (Add Tag, Apply Effect, your custom ones) will be ignored.
{% endhint %}

### Async Custom Node (Wait-on-Event Pattern)

When your node waits for something (animation, timer, external API), return `Wait()` and trigger the advance later via `UMayDialogueAsyncLibrary::RequestNodeAdvance(...)`:

```cpp
FMayDialogueTaskResult UMyDN_AwaitWebhook::ExecuteNode_Implementation(
    const FMayDialogueContext& Context)
{
    UMayDialogueInstance* Instance = Context.DialogueInstance;
    if (!Instance) return FMayDialogueTaskResult::Abort();

    const FGuid NextNode = GetFirstValidOutput(Context);

    // Register async state on the Instance — see MayDialogueNodeAsyncState.h
    UMyDN_AwaitWebhookState* State = Instance->GetOrCreateAsyncState<UMyDN_AwaitWebhookState>(NodeGuid);
    State->NextNodeGuid = NextNode;
    State->BindCompletionDelegate([Instance, NextNode]()
    {
        // UMayDialogueAsyncLibrary::RequestNodeAdvance is the public
        // Blueprint-callable wrapper (category "MayDialogue|Async"). It includes
        // a null guard and is the recommended approach even from C++.
        UMayDialogueAsyncLibrary::RequestNodeAdvance(Instance, NextNode);
    });

    return FMayDialogueTaskResult::Wait();
}
```

Don't forget delegate unbinds in `CleanupAsyncState` — otherwise your node leaks after a dialogue abort. See `MayDialogueNode_Wait.cpp` and `MayDialogueNode_PlayAnimation.cpp` as reference implementations in the plugin.

### Mixing C++ and Blueprint

When building a reusable custom node type, create it as a C++ base (`Blueprintable`) with the subsystem bridge in C++, and let designers derive concrete Blueprint subclasses (`BP_DN_QuestStart`, `BP_DN_QuestComplete`, …). This saves build cycles during iteration.

Once your C++ module is compiled, the node appears in the context menu exactly like a Blueprint node under the `Class Settings → Category` — an editor reload is sufficient, no asset refresh needed.

---

## Notes

* **Graph representation** comes automatically — your node is rendered as a standard task node (blue box). For a specialized appearance, a Slate node factory would be needed (C++, advanced).
* **Categorization** makes the difference. Without a category, the node lands under "Misc" — buried nodes don't get used.
* **Multiple outputs** are possible: add multiple GUIDs to `OutputNodeGuids`. Designers then connect multiple output pins.
