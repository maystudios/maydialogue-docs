---
description: Build your own Requirement type in Blueprint — it appears in the node palette automatically.
---

# Build a Custom Requirement in Blueprint

## Scenario

You need a condition MayDialogue does not ship: *"player carries item X in their inventory"*. Instead of translating your inventory system into a GAS tag, you build a custom Blueprint Requirement directly. It appears in the node palette automatically and behaves like any other Requirement.

## What You Will Learn

- Create a Blueprint subclass of `UMayDialogueRequirement`.
- Implement the `IsRequirementSatisfied()` function in Blueprint.
- Use `Passed`, `FailedButVisible`, `FailedAndHidden` as return values.
- Configure your own properties for the editor (inventory item name).
- Use the new Requirement type in a dialogue asset.

## Prerequisites

- [Branching with Conditions](branching-conditions.md) completed.
- An inventory system with an accessible `HasItem(FName)` function.

## What You Build

Blueprint `BPReq_HasInventoryItem` — a Requirement that checks whether the player carries a specific item. Configurable via an `ItemName` property in the Details panel.

> 📸 **Image placeholder:** `custom-blueprint-requirement-palette.png` — Node palette in the dialogue editor with the new Requirement type.
> *Setup:* PlayerChoice node in the asset editor, Requirement dropdown open. Visible in the list: the standard Requirements (HasTag, CheckAttribute, ...) and below them the new entry `Has Inventory Item (Custom)`.

## Step by Step

### 1. Create the Blueprint Class

Content Browser → **Right-click → Blueprint Class**. In the class picker, search for `MayDialogueRequirement` and select it.

Name: `BPReq_HasInventoryItem`.

### 2. Add the ItemName Property

In the Blueprint editor: **Variables → Add Variable**:
- `Variable Name`: `ItemName`
- `Variable Type`: `Name`
- `Instance Editable`: `true` (so it can be set in the dialogue editor)
- `Expose on Spawn`: `true`

> 📸 **Image placeholder:** `custom-blueprint-requirement-variable.png` — Blueprint editor with the ItemName variable.
> *Setup:* `BPReq_HasInventoryItem` Blueprint open. Variables panel on the left: `ItemName (Name, Instance Editable)` visible. Details panel on the right shows the checkbox `Instance Editable = true`.

### 3. Override the IsRequirementSatisfied Function

In the event graph: **Override → Is Requirement Satisfied**. The function receives the `Context` (an `FMayDialogueContext`) and must return an `EMayDialogueRequirementResult`. Read the instigator from the context (`Context → Instigator`).

```text
[Is Requirement Satisfied (Context)]
   │
   ▼
[Context → Instigator] → [Get Inventory Component]
   │
   ▼
[Has Item? (ItemName)]
   ├─ True  → [Return: Passed]
   └─ False → [Return: Get Fail Result (Self)]
```

Returning `Get Fail Result (Self)` on failure honors the `FailResult` property you set per-instance in the Details panel, so the same Blueprint can render greyed-out on one Choice and hidden on another.

> 📸 **Image placeholder:** `custom-blueprint-requirement-graph.png` — IsRequirementSatisfied implementation in the Blueprint graph.
> *Setup:* `BPReq_HasInventoryItem` event graph. Override `Is Requirement Satisfied` expanded. Nodes: `Context` (struct) → `Break` / `Instigator` pin → `Get Component by Class (InventoryComponent)` → `Has Item (ItemName Variable)` → Branch → two Return nodes: `Return Passed` (green) and `Return Get Fail Result` (yellow).

### 4. Use the Requirement in the Dialogue Asset

In every Requirement dropdown (Branch condition, Choice, RandomLine entry), `BPReq_HasInventoryItem` now appears automatically as an option.

Select a Choice (or a Branch condition) → **Requirements → Add → Has Inventory Item** → fill in the `ItemName` property in the Details panel:

| Property | Value |
|----------|------|
| `ItemName` | `Key_RedDoor` |
| `FailResult` | `FailedAndHidden` |

> 📸 **Image placeholder:** `custom-blueprint-requirement-in-use.png` — Details panel of a Choice with the new Requirement.
> *Setup:* Dialogue asset open, Choice selected. Details: Requirements list shows `BPReq_HasInventoryItem`, below it the property `ItemName = "Key_RedDoor"`, `FailResult = FailedAndHidden`.

### 5. Compile and Test

In PIE: start the dialogue without the item → Choice hidden. Put the item in the inventory, restart the dialogue → Choice visible.

## Return Values

| Value | Meaning |
|------|-----------|
| `Passed` | Condition met. |
| `FailedButVisible` | Not met, but the option is shown greyed out. |
| `FailedAndHidden` | Not met, the option is fully hidden. |

Prefer returning `Get Fail Result (Self)` and let the per-instance `FailResult` property decide visible vs. hidden — that way designers control it per Choice without editing the Blueprint. Use `FailedButVisible` when the player should know the option exists (e.g. *"You need a key"*); use `FailedAndHidden` for hidden paths.

## C++ Variant

For more control and performance: a C++ subclass. Override `IsRequirementSatisfied_Implementation` (the `BlueprintNativeEvent` `_Implementation` half — do **not** re-mark it `UFUNCTION`).

```cpp
UCLASS()
class UMayDlgRequirement_HasInventoryItem : public UMayDialogueRequirement
{
    GENERATED_BODY()
public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Requirement")
    FName ItemName;

    virtual EMayDialogueRequirementResult IsRequirementSatisfied_Implementation(
        const FMayDialogueContext& Context) const override
    {
        if (UInventoryComponent* Inv = Context.Instigator
            ? Context.Instigator->FindComponentByClass<UInventoryComponent>()
            : nullptr)
        {
            return Inv->HasItem(ItemName)
                ? EMayDialogueRequirementResult::Passed
                : GetFailResult();
        }
        return GetFailResult();
    }
};
```

`GetFailResult()` comes from the base class and resolves Hidden vs. Visible from the editor-configurable `FailResult` property.

## Variations / Going Further

- Multiple properties: `MinQuantity` for "at least 3 potions".
- Combine Requirements: pair it with HasTag on the same Choice (AND logic).
- Build your own SideEffects: same base-class pattern, a different function → [Extension → Custom SideEffects](../extension/custom-side-effects.md).

## Troubleshooting

**Requirement does not appear in the palette.**
The Blueprint must inherit from `UMayDialogueRequirement` (directly or indirectly). Check the class selection when creating it. Restart the editor after the first time you create it.

**`ItemName` property does not appear in the Details panel.**
`Instance Editable` is not enabled on the variable. In the Blueprint editor: select the variable → Details → checkbox `Instance Editable = true`.

**IsRequirementSatisfied always returns Passed.**
The inventory component was not found on the instigator (`Get Component by Class` returned null). Check that the player pawn has the component and that `Context.Instigator` is the player (not the NPC).
