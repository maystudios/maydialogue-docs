# GAS-Helpers

`MayDialogueGASHelpers.h` liefert drei inline-Utility-Funktionen für ASC-Resolution. Alle GAS-Klassen im Plugin nutzen diese, um Boilerplate zu vermeiden.

## Funktionen

### `GetASCFromActor(AActor* Actor)`

```cpp
inline UAbilitySystemComponent* GetASCFromActor(AActor* Actor)
```

Dual-Path:

1. `IAbilitySystemInterface::GetAbilitySystemComponent()` – der empfohlene Weg. Actor implementiert das Interface.
2. Fallback: `FindComponentByClass<UAbilitySystemComponent>()` – für Actors ohne Interface, die aber einen ASC als Component haben.

Liefert `nullptr`, wenn keiner der Wege erfolgreich ist.

### `GetTargetActor(const FMayDialogueContext& Context, bool bUseInstigator)`

```cpp
inline AActor* GetTargetActor(const FMayDialogueContext& Context, bool bUseInstigator);
```

Liefert `Context.Instigator.Get()` oder `Context.Target.Get()` je nach Flag.

### `GetASCFromContext(const FMayDialogueContext& Context, bool bUseInstigator)`

```cpp
inline UAbilitySystemComponent* GetASCFromContext(const FMayDialogueContext& Context, bool bUseInstigator);
```

Convenience-Wrapper: `GetASCFromActor(GetTargetActor(Context, bUseInstigator))`.

## Einsatz in eigenen Klassen

Wenn du eigene GAS-Requirements oder -SideEffects in C++ schreibst, nutzt du dieselben Helfer:

```cpp
#include "MayDialogueGASHelpers.h"

EMayDialogueRequirementResult UMyRequirement_HealthRange::IsRequirementSatisfied_Implementation(
    const FMayDialogueContext& Context) const
{
    UAbilitySystemComponent* ASC = MayDialogueGAS::GetASCFromContext(Context, bCheckOnInstigator);
    if (!ASC)
    {
        return EMayDialogueRequirementResult::Passed;  // Kein ASC: default-Pass.
    }

    float Health = ASC->GetNumericAttribute(UHealthAttributeSet::GetHealthAttribute());
    return (Health >= MinHealth && Health <= MaxHealth)
        ? EMayDialogueRequirementResult::Passed
        : EMayDialogueRequirementResult::FailedButVisible;
}
```

## Wie der Context kommt

Bei jedem Node-Execute wird ein `FMayDialogueContext` gebildet:

```cpp
struct FMayDialogueContext
{
    UMayDialogueInstance* DialogueInstance;
    AActor* Instigator;  // Meist der Spieler
    AActor* Target;      // Meist der NPC
    TWeakObjectPtr<UAbilitySystemComponent> InstigatorASC;  // optional, für Optimierung
    UWorld* GetWorld() const;
};
```

## Anmerkungen

* Die Helpers sind **`inline`** – keine DLL-Export-Kosten, direkte Inlining.
* Das `InstigatorASC`-Feld im Context ist **nicht immer gesetzt** – verlass dich auf die Helpers, nicht auf das Feld direkt.
* Für **Target-ASC-Cache** müsstest du selbst eine Weak-Ref halten, wenn du in einer langlebigen Struktur arbeitest.
