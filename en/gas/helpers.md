---
description: ASC resolution, target resolution — utilities used internally by all GAS classes.
---

# GAS Helpers

{% hint style="info" %}
**This page is for C++ extenders.** As a Blueprint user you don't need these helpers. The `bCheckOnInstigator` / `bAddToInstigator` etc. properties on every node are sufficient — the plugin resolves the correct ASC internally.
{% endhint %}

The `MayDialogueGAS` namespace in `MayDialogueGASHelpers.h` contains utility functions for ASC resolution. All predefined GAS Requirements and SideEffects use them internally — so you don't have to rewrite the same boilerplate when writing your own C++ classes.

---

## GetASCFromActor

```cpp
UAbilitySystemComponent* MayDialogueGAS::GetASCFromActor(AActor* Actor)
```

Finds the ASC of an actor in two steps:

1. **Preferred:** `IAbilitySystemInterface::GetAbilitySystemComponent()` — the standard path when your actor implements the interface.
2. **Fallback:** `FindComponentByClass<UAbilitySystemComponent>()` — for actors that have no direct interface path but carry an ASC as a component.

Returns `nullptr` if neither approach succeeds.

---

## GetTargetActor

```cpp
AActor* MayDialogueGAS::GetTargetActor(const FMayDialogueContext& Context, bool bUseInstigator)
```

Returns the dialogue actor to be considered:

* `bUseInstigator = true` → `Context.Instigator` (normally the player)
* `bUseInstigator = false` → `Context.Target` (normally the NPC)

---

## GetASCFromContext

```cpp
UAbilitySystemComponent* MayDialogueGAS::GetASCFromContext(const FMayDialogueContext& Context, bool bUseInstigator)
```

Convenience wrapper: combines `GetTargetActor` and `GetASCFromActor` in a single call. This is the most common entry point in custom classes.

---

## FMayDialogueContext — What's Inside

At every node execution you receive a Context:

```cpp
struct FMayDialogueContext
{
    TWeakObjectPtr<UMayDialogueInstance> DialogueInstance;
    TWeakObjectPtr<AActor> Instigator;   // Usually the player
    TWeakObjectPtr<AActor> Target;       // Usually the NPC
};
```

{% hint style="warning" %}
Rely on `GetASCFromContext()`, not directly on cached fields. The Context does not contain a directly cached `InstigatorASC` field — the helpers are the safe path.
{% endhint %}

---

## Usage in Custom C++ Classes

```cpp
#include "MayDialogueGASHelpers.h"

EMayDialogueRequirementResult UMyRequirement_HealthRange::IsRequirementSatisfied_Implementation(
    const FMayDialogueContext& Context) const
{
    UAbilitySystemComponent* ASC = MayDialogueGAS::GetASCFromContext(Context, bCheckOnInstigator);
    if (!ASC)
    {
        return EMayDialogueRequirementResult::Passed;  // No ASC: default pass
    }

    float Health = ASC->GetNumericAttribute(UHealthAttributeSet::GetHealthAttribute());
    return (Health >= MinHealth && Health <= MaxHealth)
        ? EMayDialogueRequirementResult::Passed
        : EMayDialogueRequirementResult::FailedButVisible;
}
```

> 📸 **Image placeholder:** `helpers-cpp-example.png` — Rider/VS editor with an example Requirement implementation open.
> *Setup:* File `MyRequirement_HealthRange.cpp` open in Rider. Visible: `#include "MayDialogueGASHelpers.h"`, the function body with `GetASCFromContext` call, `GetNumericAttribute` call, and return logic. Syntax highlighting active.

---

## IsDialogueServerAuthoritative

```cpp
bool MayDialogueGAS::IsDialogueServerAuthoritative(const FMayDialogueContext& Context)
```

Determines whether GAS SideEffects are allowed to run on this machine. Relevant for multiplayer:

* If the instance belongs to an actor → checks `GetLocalRole() == ROLE_Authority`.
* If the instance lives in the Subsystem → checks the roles of Instigator and Target.
* Standalone / PIE → always `true`.

All built-in SideEffects call this function internally. You only need to call it yourself when your custom C++ SideEffect makes GAS changes that should be server-exclusive.

---

## TriggerCue (Utility)

```cpp
void MayDialogueGAS::TriggerCue(UAbilitySystemComponent* ASC, const FGameplayTag& CueTag,
    const FGameplayCueParameters& Parameters, uint8 CueMode)
```

Internal helper for `UMayDlgSideEffect_TriggerCue`. Routes to the appropriate ASC method based on `CueMode` (0 = Execute, 1 = Add, 2 = Remove). You can use it in your own classes, but don't have to — a direct ASC call works just as well.

> 📸 **Image placeholder:** `helpers-header-overview.png` — `MayDialogueGASHelpers.h` open in the editor, all functions visible.
> *Setup:* File `MayDialogueGASHelpers.h` open in Rider. All four inline functions in namespace `MayDialogueGAS` visible: `GetASCFromActor`, `GetTargetActor`, `GetASCFromContext`, `IsDialogueServerAuthoritative`, `TriggerCue`. Complete signatures visible.

> 📸 **Image placeholder:** `helpers-blueprint-no-code.png` — Blueprint graph with `HasTag` SideEffect, no manual ASC call needed.
> *Setup:* Blueprint graph shows an `Event Execute Side Effect` node leading directly to a `Get Quest Subsystem` node. No `Get ASC` call visible — shows that Blueprint users never need the helpers directly.
