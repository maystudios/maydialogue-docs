# Eigene GAS-Nodes erstellen

Die drei vordefinierten Requirements und vier SideEffects decken nicht jeden Fall. Für projekt-spezifische GAS-Logik kannst du **eigene Subklassen** anlegen – entweder in C++ oder in Blueprint.

## Variante 1 – Blueprint (empfohlen für schnelle Iteration)

### Eigenes Requirement

1. Content Browser → Rechtsklick → Blueprint Class.
2. **All Classes** → `MayDialogueRequirement` auswählen.
3. Class benennen, z.B. `BP_Req_QuestCompleted`.
4. Event Graph öffnen.
5. **Override Function** → `Is Requirement Satisfied`.

Implementation:

```
Event Is Requirement Satisfied (Context):
  - Get Quest Subsystem from WorldContext
  - Is Quest "KillDragon" Completed?
    - Yes → return Passed
    - No  → return Failed (Hidden or Visible je nach bHideOnFail)
```

Sobald compiled, erscheint die Klasse im Details-Panel von Choice / Branch → **Add Requirement** → Dropdown.

### Eigener SideEffect

1. Content Browser → Rechtsklick → Blueprint Class.
2. **All Classes** → `MayDialogueSideEffect`.
3. Benennen, z.B. `BP_SE_QuestProgress`.
4. Event Graph öffnen.
5. **Override Function** → `Execute Side Effect`.

Implementation:

```
Event Execute Side Effect (Context):
  - Get Quest Subsystem
  - Increment Quest "TalkToEveryone" by 1
```

Erscheint analog im SideEffects-Array.

## Variante 2 – C++ (für enge Performance-Integration)

### Header

```cpp
// MyRequirement_ReputationInRange.h
#pragma once

#include "CoreMinimal.h"
#include "SubNodes/MayDialogueRequirement.h"
#include "MyRequirement_ReputationInRange.generated.h"

UCLASS(BlueprintType, EditInlineNew, meta=(DisplayName="Reputation In Range"))
class MYGAME_API UMyRequirement_ReputationInRange : public UMayDialogueRequirement
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, Category="Reputation")
    FGameplayTag FactionTag;

    UPROPERTY(EditAnywhere, Category="Reputation")
    int32 MinReputation = 0;

    UPROPERTY(EditAnywhere, Category="Reputation")
    int32 MaxReputation = 100;

    virtual EMayDialogueRequirementResult IsRequirementSatisfied(
        const FMayDialogueContext& Context) const override;
};
```

### Implementation

```cpp
// MyRequirement_ReputationInRange.cpp
#include "MyRequirement_ReputationInRange.h"
#include "MayDialogueGASHelpers.h"

EMayDialogueRequirementResult UMyRequirement_ReputationInRange::IsRequirementSatisfied(
    const FMayDialogueContext& Context) const
{
    if (!FactionTag.IsValid())
    {
        return EMayDialogueRequirementResult::Passed;
    }

    UAbilitySystemComponent* ASC = MayDialogueGAS::GetASCFromContext(Context, /*bUseInstigator=*/true);
    if (!ASC)
    {
        return EMayDialogueRequirementResult::Passed;
    }

    // Beispiel: Attribut je Fraktion per Tag-Lookup in deinem AttributeSet
    float Rep = GetReputationForFaction(ASC, FactionTag);
    return (Rep >= MinReputation && Rep <= MaxReputation)
        ? EMayDialogueRequirementResult::Passed
        : (bHideOnFail
            ? EMayDialogueRequirementResult::FailedAndHidden
            : EMayDialogueRequirementResult::FailedButVisible);
}
```

## Wo hinzufügen?

C++ oder Blueprint – egal: sobald die Klasse existiert und kompiliert ist, erscheint sie im Details-Panel jedes Nodes, der Requirements / SideEffects akzeptiert.

## Best Practices

* **Eine Sache pro Klasse**. Wenn dein Requirement „HasAbility AND HasTag" prüft, mach zwei separate Requirements – Designer kann beide auf denselben Choice legen.
* **Description sinnvoll befüllen** (`FText Description`) – es erscheint als Pill-Label im Graph.
* **GetDisplayDescription überschreiben**, wenn der Default-Text nicht selbsterklärend ist.
* **Fail-Modi respektieren** – `bHideOnFail` und `FailureResult` sollten vom Designer steuerbar bleiben.
* **Context prüfen** – Instigator/Target können `nullptr` sein. Default-Pass bei Unklarheit, nicht Crash.

## Test-Workflow

1. Klasse bauen.
2. Compile.
3. Dialog-Asset öffnen.
4. Requirement/SideEffect am Choice hinzufügen.
5. Dialog compilen und im Preview-Runner testen.
6. Im PIE mit Debugger & Watch-Panel validieren.
