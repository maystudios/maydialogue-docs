---
description: C++-Setup für Plugin-Erweiterungen — Build.cs, BlueprintNativeEvent-Pattern, Module-Reload.
---

# C++-Erweiterung — Grundlagen

Du kannst MayDialogue auf zwei Wegen erweitern: Blueprint (für Designer-Workflows) und C++ (für komplexe Logik, performance-kritische Pfade, Source-Control-freundliche Diffs). Beide Pfade sind voll unterstützt — diese Seite deckt das C++-Setup ab, das für **alle** drei Erweiterungs-Typen (Requirements, SideEffects, Nodes) gleich ist.

---

## 1. Module-Dependency in deiner Build.cs

In `<DeinModul>/<DeinModul>.Build.cs` `MayDialogue` als Public-Dependency hinzufügen:

```csharp
PublicDependencyModuleNames.AddRange(new string[] {
    "Core", "CoreUObject", "Engine",
    "MayDialogue",          // Runtime-Module (Requirements, SideEffects, Nodes, Subsystem)
    "GameplayTags",         // wird von MayDialogue-Typen referenziert
});

// Optional — falls du GAS-Helpers oder die mitgelieferten GAS-Requirements/SideEffects nutzt:
PublicDependencyModuleNames.Add("MayDialogueGAS");

// Optional — falls deine C++-Klassen im Editor Sonderdarstellung brauchen (Slate-Node-Factory, Details-Customization):
if (Target.Type == TargetType.Editor)
{
    PrivateDependencyModuleNames.Add("MayDialogueEditor");
}
```

> **Editor-Module nur als Editor-Dependency.** `MayDialogueEditor` darf NIE in einem Runtime/Game-Target gelistet werden — das bricht den Shipping-Build. Verpacke es in `if (Target.Type == TargetType.Editor)` oder in eine eigene `WhitelistPlatforms = ...` Logik.

---

## 2. BlueprintNativeEvent — was, wie, warum

Die drei Plugin-Basisklassen exposen ihre überschreibbaren Funktionen mit `UFUNCTION(BlueprintNativeEvent)`. Das heißt:

* Eine **C++-Default-Implementation** existiert in der Basisklasse (suffix `_Implementation`).
* Ein **Blueprint-Override** ist optional — wenn die Subklasse BP ist und die Funktion überschreibt, wird die BP-Version aufgerufen.
* Eine **C++-Subklasse** überschreibt `virtual <Name>_Implementation(...) override;` — kein Override des Wrapper-Symbols, sondern der `_Implementation`.

Konkret in `UMayDialogueRequirement`:

```cpp
// Header (Plugin)
UFUNCTION(BlueprintNativeEvent, Category = "MayDialogue|Requirement")
EMayDialogueRequirementResult IsRequirementSatisfied(const FMayDialogueContext& Context) const;
virtual EMayDialogueRequirementResult IsRequirementSatisfied_Implementation(const FMayDialogueContext& Context) const;
```

In deinem C++-Subclass:

```cpp
virtual EMayDialogueRequirementResult IsRequirementSatisfied_Implementation(
    const FMayDialogueContext& Context) const override;
```

> **Warum nicht `BlueprintImplementableEvent`?** Das gibt es zwar in UE, hat aber **keinen** C++-Default und **erzwingt** eine BP-Override. Sobald du eine reine C++-Subklasse machst (oder eine BP-Subklasse die Override-Funktion gar nicht überschreibt), wird ein BlueprintImplementableEvent zum stillen No-Op. `BlueprintNativeEvent` ist bei extensiblen Plugin-Hooks immer die richtige Wahl.

---

## 3. Header-Struktur einer Subklasse

Minimale Header-Pflicht: `CoreMinimal.h` zuerst, das Basis-Header von MayDialogue, dann der `.generated.h` als allerletzter Include.

```cpp
// MyReq_QuestCompleted.h
#pragma once

#include "CoreMinimal.h"
#include "SubNodes/MayDialogueRequirement.h"   // Basis-Klasse + EMayDialogueRequirementResult
#include "MyReq_QuestCompleted.generated.h"

struct FMayDialogueContext;                     // Forward-decl für Function-Signaturen

UCLASS(BlueprintType, EditInlineNew, meta=(DisplayName="Quest Completed"))
class MYGAME_API UMyReq_QuestCompleted : public UMayDialogueRequirement
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Quest")
    FName QuestID;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Quest")
    int32 RequiredStage = 0;

    // BlueprintNativeEvent-Override: KEIN UFUNCTION-Macro hier, NUR override des _Implementation.
    virtual EMayDialogueRequirementResult IsRequirementSatisfied_Implementation(
        const FMayDialogueContext& Context) const override;

    virtual FText GetDisplayDescription_Implementation() const override;
};
```

> **`EditInlineNew` ist Pflicht für Sub-Nodes.** Ohne dieses Specifier kann der Editor deine Klasse nicht inline in den Requirements/SideEffects-Array eines Nodes anlegen — sie würde nur als separates Asset funktionieren, was nicht gewollt ist.
> **`Blueprintable` weglassen** wenn du nur diese eine konkrete Klasse willst und keine weiteren BP-Subklassen darauf basieren sollen. Falls Designer von deinem C++ aus weiter ableiten sollen (Inheritance-Chain), füg `Blueprintable` hinzu.

---

## 4. Implementation-File

```cpp
// MyReq_QuestCompleted.cpp
#include "MyReq_QuestCompleted.h"
#include "Instance/MayDialogueInstance.h"
#include "Types/MayDialogueTypes.h"             // FMayDialogueContext-Definition
#include "MyGame/Quest/QuestSubsystem.h"        // dein Projekt-Subsystem

EMayDialogueRequirementResult UMyReq_QuestCompleted::IsRequirementSatisfied_Implementation(
    const FMayDialogueContext& Context) const
{
    if (QuestID == NAME_None)
    {
        return EMayDialogueRequirementResult::Passed;     // leere Konfig = nicht blockierend
    }

    UWorld* World = Context.DialogueInstance ? Context.DialogueInstance->GetWorld() : nullptr;
    UQuestSubsystem* Quest = World ? UQuestSubsystem::Get(World) : nullptr;
    if (!Quest)
    {
        return EMayDialogueRequirementResult::Passed;     // Subsystem fehlt = degrade zu OK
    }

    if (Quest->IsCompletedAtStage(QuestID, RequiredStage))
    {
        return EMayDialogueRequirementResult::Passed;
    }

    // Fail-Modus aus der Basisklasse honorieren — siehe FailResult-Property auf UMayDialogueRequirement.
    return GetFailResult();
}

FText UMyReq_QuestCompleted::GetDisplayDescription_Implementation() const
{
    return FText::Format(
        NSLOCTEXT("MyGame", "QuestCompletedDesc", "Quest abgeschlossen: {0} (Stage ≥ {1})"),
        FText::FromName(QuestID),
        FText::AsNumber(RequiredStage));
}
```

`GetFailResult()` kommt aus der Basisklasse und resolved Hidden vs Visible aus dem Editor-konfigurierbaren `FailResult`-Property (plus dem deprecated `bHideOnFail`-Flag für Migration).

---

## 5. Module-Reload und Editor-Sichtbarkeit

Nach dem Bauen erscheint deine C++-Klasse **automatisch** im Editor-Class-Picker:

1. Build aus Rider/VS oder per Build.bat. Editor neu starten (oder Live-Coding mit Strg+Alt+F11 versuchen).
2. Im Dialog-Asset eine Choice oder einen Node auswählen.
3. Details-Panel → **Add Requirement** (oder **Add SideEffect**) → dein C++-Klassenname erscheint im Dropdown unter `meta=(DisplayName)` Wert.
4. Inline-Properties sichtbar im Details-Panel.

Für komplett neue Node-Typen (`MayDialogueNode_Base`-Subclasses) gilt analog: Rechtsklick im Graph → Kontext-Menü zeigt deine Klasse unter der `Class Settings → Category` — siehe [Custom Nodes](custom-nodes.md).

---

## 6. Live-Coding-Caveats

Live Coding (Strg+Alt+F11) funktioniert für **Methoden-Bodies**. Es funktioniert NICHT für:

* Neue UPROPERTY-Felder (Reflection-Schema-Änderung)
* Neue UFUNCTION-Members
* Neue UCLASSes
* Änderungen am USTRUCT-Layout

Für solche Änderungen: **Editor schließen → Build → Editor neu öffnen**. Andernfalls riskierst du Cooked-Asset-Inkonsistenzen oder Crashes beim Laden.

---

## 7. Wann Blueprint, wann C++?

| Kriterium | Blueprint | C++ |
| --- | --- | --- |
| Designer-iterierbar | ✅ | ❌ (Build-Cycle nötig) |
| Performance-kritischer Hot-Path | ❌ | ✅ |
| Komplexe Subsystem-Logik | ⚠️ schwer lesbar | ✅ |
| Source-Control-Diffs | ⚠️ Binary | ✅ Text |
| Cross-Class-Refactoring | ❌ schmerzhaft | ✅ IDE-Tools |
| 1:1-Reproduzierbarkeit | ⚠️ Asset-Korruption möglich | ✅ |
| Schnelles Prototyping | ✅ | ❌ |

**Empfehlung:** Designer-facing Logik (Quest-Bedingungen, Inventory-Checks, Reputation-Gates) als BP. Engine-/Subsystem-Bridges, performance-kritische Tick-Pfade, Logik mit komplexer State-Machine als C++.

Du kannst ein **C++-Basis + BP-Override** mischen: leg eine abstrakte C++-Basis mit `Blueprintable` an, mach einen oder zwei BP-Subclasses, und Designer leiten von deinen BPs ab. Best-of-both-worlds.

---

## Querverweise

* [Eigene Requirements](custom-requirements.md) — komplettes Beispiel inkl. C++-Variante
* [Eigene SideEffects](custom-side-effects.md) — Server/Client-Split-Pattern
* [Eigene Nodes](custom-nodes.md) — Custom Task-Nodes inkl. Async-Pattern
* [Bridge implementieren](bridge-implementation.md) — Subsystem-Integration für externe Quest-/Save-Systeme
