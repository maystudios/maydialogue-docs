---
description: Custom GAS Requirements and Action Nodes via Blueprint — step by step.
---

# Creating Custom GAS Nodes

The three built-in Requirements and four SideEffects don't cover every project. For project-specific GAS logic, you create your own subclasses — entirely in Blueprint, no C++ required.

---

## Custom GAS Requirement (Blueprint)

Example goal: a Choice appears only when a specific quest has been completed.

### Step 1 — Create Blueprint Class

1. Content Browser → right-click → **Blueprint Class**.
2. Open **All Classes** in the class picker.
3. Type `MayDialogueRequirement` and select it.
4. Name the class, e.g. `BP_Req_QuestCompleted`.

> 📸 **Image placeholder:** `gasext-req-create-class.png` — "Pick Parent Class" dialog with `MayDialogueRequirement` in the search.
> *Setup:* "All Classes" tab active. Search field contains "MayDialogueRequirement". Result list shows `UMayDialogueRequirement` with full path. Mouse cursor on the entry.

### Step 2 — Add Properties

In the Blueprint editor **Variables panel**: create a new variable.

* `QuestID` — type `FName`, **Instance Editable = true**, **Expose on Spawn = false**
* Optional: `bRequireCompleted` — type `bool`, Instance Editable = true

Instance Editable ensures that the designer can enter the value directly in the node's Details panel.

> 📸 **Image placeholder:** `gasext-req-variables.png` — Variables panel of the Blueprint with the `QuestID` entry.
> *Setup:* Blueprint editor open, Variables panel on the left. `QuestID` of type `Name` selected. On the right in the Details panel: `Instance Editable = true` (checked), `Category = "Quest"`.

### Step 3 — Override IsRequirementSatisfied

In the Functions panel → **Override** → `Is Requirement Satisfied`.

Pseudo-graph:

```text
Event Is Requirement Satisfied (Context)
  │
  ├─ Get Quest Subsystem (from Context → Get World)
  │
  ├─ Is Quest Completed? (QuestID)
  │     ├─ true  → Return Passed
  │     └─ false → Branch: bHideOnFail?
  │                   ├─ true  → Return FailedAndHidden
  │                   └─ false → Return FailedButVisible
```

{% hint style="info" %}
`bHideOnFail` is already defined in the base class `UMayDialogueRequirement`. You don't need to create it yourself — it automatically appears in the Details panel of every subclass.
{% endhint %}

> 📸 **Image placeholder:** `gasext-req-bp-graph.png` — Fully implemented `Is Requirement Satisfied` graph.
> *Setup:* Event `IsRequirementSatisfied` open in the Blueprint. From left: `Event` node → `Get World` → `Get Quest Subsystem` → `Is Quest Completed (QuestID)` → Branch node. True path: `Return Passed`. False path: another branch on `bHideOnFail` → two Returns (`FailedAndHidden` / `FailedButVisible`). All connections visible.

### Step 4 — Test

1. Compile the Blueprint (Ctrl+F9).
2. Open a dialogue asset.
3. Click on a Choice → Details panel → **Add Requirement** → Dropdown → select `BP_Req_QuestCompleted`.
4. Fill in the `QuestID` field.
5. Compile the dialogue, test in the Preview Runner.

---

## Custom GAS SideEffect (Blueprint)

Example goal: when entering a SayLine, quest progress should increase by +1.

### Step 1 — Create Blueprint Class

1. Content Browser → right-click → **Blueprint Class**.
2. Select `MayDialogueSideEffect` as parent.
3. Name it: `BP_SE_QuestProgressUpdate`.

> 📸 **Image placeholder:** `gasext-se-create-class.png` — "Pick Parent Class" dialog with `MayDialogueSideEffect`.
> *Setup:* Same as Requirement Step 1, but `MayDialogueSideEffect` in the search field. Result shows `UMayDialogueSideEffect`.

### Step 2 — Add Properties

* `QuestID` — FName, Instance Editable = true
* `ProgressDelta` — int32, Instance Editable = true, Default value = 1

### Step 3 — Override ExecuteSideEffect

Functions panel → **Override** → `Execute Side Effect`.

Pseudo-graph:

```text
Event Execute Side Effect (Context)
  │
  ├─ Get Quest Subsystem (from Context → Get World)
  │
  └─ Add Quest Progress (QuestID, ProgressDelta)
```

> 📸 **Image placeholder:** `gasext-se-bp-graph.png` — Execute Side Effect graph with Quest Progress call.
> *Setup:* `Event ExecuteSideEffect` → `Get World` → `Get Quest Subsystem` → `Add Quest Progress` with pinned `QuestID` and `ProgressDelta` variables. All pins connected, Blueprint compiled (no error symbol).

### Step 4 — Use in a Dialogue

On a SayLine in the Details panel → **SideEffects** → **+** → `BP_SE_QuestProgressUpdate` → fill in QuestID and ProgressDelta.

---

## Best Practices

* **One thing per class.** Prefer `BP_Req_QuestCompleted` and `BP_Req_HasEnoughGold` over a monolithic "AllConditions" Requirement.
* **Add null guards.** Check whether your subsystem was found — return `Passed` when the subsystem is missing, not `FailedAndHidden`. Otherwise all Choices disappear if your subsystem doesn't load.
* **Use display names.** Enter a meaningful **Blueprint Node Title** in Class Settings — this appears as the pill label in the graph.
* **Always navigate via `GetWorld()` from the Context.** Don't point directly to `GWorld`.

---

{% hint style="success" %}
**C++ Variant**

If you want to work in C++ for performance or type safety reasons:

**Requirement:**
```cpp
UCLASS(BlueprintType, EditInlineNew, meta=(DisplayName="Quest Completed"))
class MYGAME_API UMyReq_QuestCompleted : public UMayDialogueRequirement
{
    GENERATED_BODY()
public:
    UPROPERTY(EditAnywhere, Category="Quest")
    FName QuestID;

    virtual EMayDialogueRequirementResult IsRequirementSatisfied_Implementation(
        const FMayDialogueContext& Context) const override;
};
```

```cpp
EMayDialogueRequirementResult UMyReq_QuestCompleted::IsRequirementSatisfied_Implementation(
    const FMayDialogueContext& Context) const
{
    if (QuestID == NAME_None) return EMayDialogueRequirementResult::Passed;

    UWorld* World = Context.DialogueInstance ? Context.DialogueInstance->GetWorld() : nullptr;
    auto* Quest = World ? UQuestSubsystem::Get(World) : nullptr;
    if (!Quest) return EMayDialogueRequirementResult::Passed;

    return Quest->IsCompleted(QuestID)
        ? EMayDialogueRequirementResult::Passed
        : (bHideOnFail
            ? EMayDialogueRequirementResult::FailedAndHidden
            : EMayDialogueRequirementResult::FailedButVisible);
}
```

**SideEffect:**
```cpp
UCLASS(BlueprintType, EditInlineNew, meta=(DisplayName="Quest Progress Update"))
class MYGAME_API UMySE_QuestProgressUpdate : public UMayDialogueSideEffect
{
    GENERATED_BODY()
public:
    UPROPERTY(EditAnywhere, Category="Quest") FName QuestID;
    UPROPERTY(EditAnywhere, Category="Quest") int32 ProgressDelta = 1;

    virtual void ExecuteSideEffect_Implementation(const FMayDialogueContext& Context) override;
};
```

```cpp
void UMySE_QuestProgressUpdate::ExecuteSideEffect_Implementation(const FMayDialogueContext& Context)
{
    if (QuestID == NAME_None) return;
    if (auto* Quest = UQuestSubsystem::Get(Context.DialogueInstance->GetWorld()))
    {
        Quest->AddProgress(QuestID, ProgressDelta);
    }
}
```

Both classes appear automatically in the node pickers once the project is compiled.
{% endhint %}
