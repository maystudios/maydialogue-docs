---
description: C++ setup for plugin extensions — Build.cs, BlueprintNativeEvent pattern, module reload.
---

# C++ Extension — Fundamentals

You can extend MayDialogue in two ways: Blueprint (for designer workflows) and C++ (for complex logic, performance-critical paths, source-control-friendly diffs). Both paths are fully supported — this page covers the C++ setup that is the same for **all** three extension types (Requirements, SideEffects, Nodes).

---

## 1. Module Dependency in Your Build.cs

Add `MayDialogue` as a public dependency in `<YourModule>/<YourModule>.Build.cs`:

```csharp
PublicDependencyModuleNames.AddRange(new string[] {
    "Core", "CoreUObject", "Engine",
    "MayDialogue",          // Runtime module (Requirements, SideEffects, Nodes, Subsystem)
    "GameplayTags",         // Referenced by MayDialogue types
});

// Optional — if you use GAS helpers or the included GAS Requirements/SideEffects:
PublicDependencyModuleNames.Add("MayDialogueGAS");

// Optional — if your C++ classes need special editor representation (Slate node factory, Details customization):
if (Target.Type == TargetType.Editor)
{
    PrivateDependencyModuleNames.Add("MayDialogueEditor");
}
```

> **Editor module as editor-only dependency.** `MayDialogueEditor` must NEVER be listed in a Runtime/Game target — it breaks the shipping build. Wrap it in `if (Target.Type == TargetType.Editor)` or use `WhitelistPlatforms` logic.

---

## 2. BlueprintNativeEvent — What, How, Why

The three plugin base classes expose their overridable functions with `UFUNCTION(BlueprintNativeEvent)`. This means:

* A **C++ default implementation** exists in the base class (suffix `_Implementation`).
* A **Blueprint override** is optional — if the subclass is a Blueprint and overrides the function, the Blueprint version is called.
* A **C++ subclass** overrides `virtual <Name>_Implementation(...) override;` — not the wrapper symbol, but the `_Implementation`.

Concretely in `UMayDialogueRequirement`:

```cpp
// Header (Plugin)
UFUNCTION(BlueprintNativeEvent, Category = "MayDialogue|Requirement")
EMayDialogueRequirementResult IsRequirementSatisfied(const FMayDialogueContext& Context) const;
virtual EMayDialogueRequirementResult IsRequirementSatisfied_Implementation(const FMayDialogueContext& Context) const;
```

In your C++ subclass:

```cpp
virtual EMayDialogueRequirementResult IsRequirementSatisfied_Implementation(
    const FMayDialogueContext& Context) const override;
```

> **Why not `BlueprintImplementableEvent`?** That exists in UE, but has **no** C++ default and **forces** a Blueprint override. As soon as you make a pure C++ subclass (or a Blueprint subclass that doesn't override the function at all), a BlueprintImplementableEvent becomes a silent no-op. `BlueprintNativeEvent` is always the right choice for extensible plugin hooks.

---

## 3. Header Structure of a Subclass

Minimum header requirements: `CoreMinimal.h` first, the MayDialogue base header, then `.generated.h` as the very last include.

```cpp
// MyReq_QuestCompleted.h
#pragma once

#include "CoreMinimal.h"
#include "SubNodes/MayDialogueRequirement.h"   // Base class + EMayDialogueRequirementResult
#include "MyReq_QuestCompleted.generated.h"

struct FMayDialogueContext;                     // Forward decl for function signatures

UCLASS(BlueprintType, EditInlineNew, meta=(DisplayName="Quest Completed"))
class MYGAME_API UMyReq_QuestCompleted : public UMayDialogueRequirement
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Quest")
    FName QuestID;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Quest")
    int32 RequiredStage = 0;

    // BlueprintNativeEvent override: NO UFUNCTION macro here, only override the _Implementation.
    virtual EMayDialogueRequirementResult IsRequirementSatisfied_Implementation(
        const FMayDialogueContext& Context) const override;

    virtual FText GetDisplayDescription_Implementation() const override;
};
```

> **`EditInlineNew` is required for sub-nodes.** Without this specifier, the editor cannot create your class inline in a node's Requirements/SideEffects array — it would only work as a separate asset, which is not the intended behavior.
> **Omit `Blueprintable`** if you only want this one concrete class and no further Blueprint subclasses should be derived from it. If designers need to further subclass from your C++ class (inheritance chain), add `Blueprintable`.

---

## 4. Implementation File

```cpp
// MyReq_QuestCompleted.cpp
#include "MyReq_QuestCompleted.h"
#include "Instance/MayDialogueInstance.h"
#include "Types/MayDialogueTypes.h"             // FMayDialogueContext definition
#include "MyGame/Quest/QuestSubsystem.h"        // your project subsystem

EMayDialogueRequirementResult UMyReq_QuestCompleted::IsRequirementSatisfied_Implementation(
    const FMayDialogueContext& Context) const
{
    if (QuestID == NAME_None)
    {
        return EMayDialogueRequirementResult::Passed;     // empty config = non-blocking
    }

    UWorld* World = Context.DialogueInstance ? Context.DialogueInstance->GetWorld() : nullptr;
    UQuestSubsystem* Quest = World ? UQuestSubsystem::Get(World) : nullptr;
    if (!Quest)
    {
        return EMayDialogueRequirementResult::Passed;     // subsystem missing = degrade to OK
    }

    if (Quest->IsCompletedAtStage(QuestID, RequiredStage))
    {
        return EMayDialogueRequirementResult::Passed;
    }

    // Honor the fail mode from the base class — see FailResult property on UMayDialogueRequirement.
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

`GetFailResult()` comes from the base class and resolves Hidden vs. Visible from the editor-configurable `FailResult` property (plus the deprecated `bHideOnFail` flag for migration).

---

## 5. Blueprint Designer Libraries (Overview)

Three ready-made Function Libraries are available to Blueprint designers, enabling them to get started without linking a C++ module:

| Library | Module | Methods |
| --- | --- | --- |
| `UMayDialogueGASLibrary` | `MayDialogueGAS` | `Is Dialogue Server Authoritative`, `Get ASC From Context`, `Trigger Cue On Context` — GAS helpers without manual ASC casts |
| `UMayDialogueAsyncLibrary` | `MayDialogue` | `Request Node Advance` (Blueprint async pattern), `Evaluate All Requirements`, `Execute All Side Effects`, `Execute All Client Side Effects` |
| `UMayDialogueVariablesLibrary` | `MayDialogue` | `Get Dialogue Variable`, `Set Dialogue Variable`, `Copy Dialogue Variables` — wrappers on the `FMayDialogueVariables` container |

Additionally on `UMayDialogueLibrary` (existing library): eight `Make *` nodes for `FMayDialogueTaskResult` — `Make Advance`, `Make Abort`, `Make Wait`, `Make Pause And Present Choices`, `Make Advance With Choice`, `Make Return To Start`, `Make Return To Last`, `Make Return To Current`. These Make nodes are the recommended way to return from `ExecuteNode` in a Custom Node Blueprint.

C++ developers use the static factories directly (`FMayDialogueTaskResult::Advance(...)` etc.) — the libraries are a pure Blueprint convenience layer.

---

## 6. Module Reload and Editor Visibility

After building, your C++ class appears **automatically** in the editor class picker:

1. Build from Rider/VS or via Build.bat. Restart the editor (or try Live Coding with Ctrl+Alt+F11).
2. Open a dialogue asset, select a Choice or Node.
3. Details panel → **Add Requirement** (or **Add SideEffect**) → your C++ class name appears in the dropdown under its `meta=(DisplayName)` value.
4. Inline properties visible in the Details panel.

For entirely new node types (`MayDialogueNode_Base` subclasses) the same applies: right-click in the graph → the context menu shows your class under the `Class Settings → Category` — see [Custom Nodes](custom-nodes.md).

---

## 7. Live Coding Caveats

Live Coding (Ctrl+Alt+F11) works for **method bodies**. It does NOT work for:

* New UPROPERTY fields (reflection schema change)
* New UFUNCTION members
* New UCLASSes
* Changes to USTRUCT layout

For such changes: **close the editor → build → reopen the editor**. Otherwise you risk cooked asset inconsistencies or crashes on load.

---

## 8. When Blueprint, When C++?

| Criterion | Blueprint | C++ |
| --- | --- | --- |
| Designer-iterable | ✅ | ❌ (build cycle required) |
| Performance-critical hot path | ❌ | ✅ |
| Complex subsystem logic | ⚠️ hard to read | ✅ |
| Source control diffs | ⚠️ Binary | ✅ Text |
| Cross-class refactoring | ❌ painful | ✅ IDE tools |
| 1:1 reproducibility | ⚠️ asset corruption possible | ✅ |
| Rapid prototyping | ✅ | ❌ |

**Recommendation:** Designer-facing logic (quest conditions, inventory checks, reputation gates) as Blueprint. Engine/subsystem bridges, performance-critical tick paths, logic with complex state machines as C++.

You can mix **C++ base + Blueprint override**: create an abstract C++ base with `Blueprintable`, make one or two Blueprint subclasses, and let designers derive from your Blueprints. Best of both worlds.

---

## Cross-References

* [Custom Requirements](custom-requirements.md) — complete example including C++ variant
* [Custom SideEffects](custom-side-effects.md) — server/client split pattern
* [Custom Nodes](custom-nodes.md) — custom task nodes including async pattern
* [Bridge Implementation](bridge-implementation.md) — subsystem integration for external quest/save systems
