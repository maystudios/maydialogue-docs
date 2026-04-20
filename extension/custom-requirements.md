# Eigene Requirements

Der häufigste Erweiterungs-Fall: projekt-spezifische Bedingungen. Siehe auch [GAS → Eigene GAS-Nodes](../gas/extending.md) für die GAS-bezogenen Varianten.

## Blueprint-Variante

1. Content Browser → Rechtsklick → Blueprint Class → `MayDialogueRequirement` als Parent.
2. Benennen, z.B. `BP_Req_QuestActive`.
3. Event Graph: **Override** → `Is Requirement Satisfied`.

```
Event Is Requirement Satisfied (Context):
  - Get Quest Subsystem from Context
  - Is Quest "DragonQuest" In Progress?
  - If true: return Passed
  - If false and bHideOnFail: return FailedAndHidden
  - Else: return FailedButVisible
```

### Properties

Im Variables-Panel:

* `QuestID: FName`
* `RequiredStage: int32`

**Instance Editable = true**, damit Designer sie pro Node setzen kann.

## C++-Variante

```cpp
UCLASS(BlueprintType, EditInlineNew, meta=(DisplayName="Quest Active"))
class MYGAME_API UMyReq_QuestActive : public UMayDialogueRequirement
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, Category="Quest")
    FName QuestID;

    UPROPERTY(EditAnywhere, Category="Quest")
    int32 RequiredStage = 0;

    virtual EMayDialogueRequirementResult IsRequirementSatisfied(const FMayDialogueContext& Context) const override;
    virtual FText GetDisplayDescription() const override;
};
```

```cpp
EMayDialogueRequirementResult UMyReq_QuestActive::IsRequirementSatisfied(const FMayDialogueContext& Context) const
{
    if (QuestID == NAME_None) return EMayDialogueRequirementResult::Passed;

    auto* Quest = UQuestSubsystem::Get(Context.GetWorld());
    if (!Quest) return EMayDialogueRequirementResult::Passed;

    if (Quest->IsQuestActiveAtStage(QuestID, RequiredStage))
    {
        return EMayDialogueRequirementResult::Passed;
    }

    return bHideOnFail
        ? EMayDialogueRequirementResult::FailedAndHidden
        : EMayDialogueRequirementResult::FailedButVisible;
}

FText UMyReq_QuestActive::GetDisplayDescription() const
{
    return FText::Format(
        NSLOCTEXT("MayDialogue", "QuestActiveDesc", "Quest {0} at stage {1}"),
        FText::FromName(QuestID),
        FText::AsNumber(RequiredStage)
    );
}
```

## Composable Requirements

Requirements können auf derselben Choice kombiniert werden. Das System merged alle Ergebnisse:

* Alle Passed → Passed.
* Irgendeiner FailedAndHidden → FailedAndHidden.
* Sonst irgendeiner FailedButVisible → FailedButVisible.

**Empfehlung**: Ein Requirement prüft **eine Bedingung**. Wenn du „HasQuest + Reputation >= 50 + HasTag" brauchst, nutze drei separate Sub-Nodes.

## Design-Tipps

* **`Description`-Feld nutzen**: Wird als Pill-Label im Graph gezeigt.
* **Fail-Modus konfigurierbar**: Nutze `bHideOnFail` (base) oder ein projekt-spezifisches `FailureResult`-Feld wie bei `CheckAttribute`.
* **Kein Crash bei fehlenden Services**: Default-Pass ist meist sinnvoll, wenn keine Projekt-Systeme ansprechbar sind.

## Typische projekt-spezifische Requirements

* `Req_QuestActive` — wie oben.
* `Req_QuestCompleted`.
* `Req_InventoryContains(ItemID, MinCount)`.
* `Req_TimeOfDay(MinHour, MaxHour)`.
* `Req_AreaVisited(AreaTag)`.
* `Req_NPCRelationship(NPCTag, MinLevel)`.

Jedes davon passt in einen Blueprint, den der Designer direkt im Details-Panel auswählt.
