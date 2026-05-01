---
description: ASC-Auflösung, Target-Resolution — Utilities, die alle GAS-Klassen intern nutzen.
---

# GAS-Helpers

{% hint style="info" %}
**Diese Seite richtet sich an C++-Erweiterer.** Als Blueprint-Nutzer brauchst du diese Helfer nicht. Die Properties `bCheckOnInstigator` / `bAddToInstigator` etc. an jedem Node reichen — das Plugin löst den richtigen ASC intern auf.
{% endhint %}

Der Namespace `MayDialogueGAS` in `MayDialogueGASHelpers.h` enthält Hilfsfunktionen für die ASC-Auflösung. Alle vordefinierten GAS-Requirements und -SideEffects nutzen sie intern — damit du beim Schreiben eigener C++-Klassen nicht dasselbe Boilerplate neu schreiben musst.

---

## GetASCFromActor

```cpp
UAbilitySystemComponent* MayDialogueGAS::GetASCFromActor(AActor* Actor)
```

Sucht den ASC eines Actors in zwei Schritten:

1. **Bevorzugt:** `IAbilitySystemInterface::GetAbilitySystemComponent()` — der Standard-Weg, wenn dein Actor das Interface implementiert.
2. **Fallback:** `FindComponentByClass<UAbilitySystemComponent>()` — für Actors, die keinen direkten Interface-Pfad haben, aber einen ASC als Komponente tragen.

Gibt `nullptr` zurück, wenn keiner der Wege erfolgreich ist.

---

## GetTargetActor

```cpp
AActor* MayDialogueGAS::GetTargetActor(const FMayDialogueContext& Context, bool bUseInstigator)
```

Liefert den Dialog-Actor, der betrachtet werden soll:

* `bUseInstigator = true` → `Context.Instigator` (normalerweise der Spieler)
* `bUseInstigator = false` → `Context.Target` (normalerweise der NPC)

---

## GetASCFromContext

```cpp
UAbilitySystemComponent* MayDialogueGAS::GetASCFromContext(const FMayDialogueContext& Context, bool bUseInstigator)
```

Convenience-Wrapper: Kombiniert `GetTargetActor` und `GetASCFromActor` in einem Aufruf. Das ist der häufigste Einstiegspunkt in eigenen Klassen.

---

## FMayDialogueContext — was drin steckt

Bei jedem Node-Execute bekommst du einen Context:

```cpp
struct FMayDialogueContext
{
    TWeakObjectPtr<UMayDialogueInstance> DialogueInstance;
    TWeakObjectPtr<AActor> Instigator;   // Meist der Spieler
    TWeakObjectPtr<AActor> Target;       // Meist der NPC
};
```

{% hint style="warning" %}
Verlass dich auf `GetASCFromContext()`, nicht direkt auf gecachte Felder. Der Context enthält kein direkt gecachtes `InstigatorASC`-Feld — die Helpers sind der sichere Weg.
{% endhint %}

---

## Einsatz in eigenen C++-Klassen

```cpp
#include "MayDialogueGASHelpers.h"

EMayDialogueRequirementResult UMyRequirement_HealthRange::IsRequirementSatisfied_Implementation(
    const FMayDialogueContext& Context) const
{
    UAbilitySystemComponent* ASC = MayDialogueGAS::GetASCFromContext(Context, bCheckOnInstigator);
    if (!ASC)
    {
        return EMayDialogueRequirementResult::Passed;  // Kein ASC: default-Pass
    }

    float Health = ASC->GetNumericAttribute(UHealthAttributeSet::GetHealthAttribute());
    return (Health >= MinHealth && Health <= MaxHealth)
        ? EMayDialogueRequirementResult::Passed
        : EMayDialogueRequirementResult::FailedButVisible;
}
```

> 📸 **Bild-Platzhalter:** `helpers-cpp-example.png` — Rider/VS-Editor mit geöffneter Beispiel-Requirement-Implementierung.
> *Setup:* Die Datei `MyRequirement_HealthRange.cpp` in Rider öffnen. Sichtbar: `#include "MayDialogueGASHelpers.h"`, der Funktionskörper mit `GetASCFromContext`-Aufruf, `GetNumericAttribute`-Aufruf und Return-Logik. Syntax-Highlighting aktiv.

---

## IsDialogueServerAuthoritative

```cpp
bool MayDialogueGAS::IsDialogueServerAuthoritative(const FMayDialogueContext& Context)
```

Bestimmt, ob GAS-SideEffects auf diesem Gerät laufen dürfen. Relevant für Multiplayer:

* Wenn die Instance einem Actor gehört → prüft `GetLocalRole() == ROLE_Authority`.
* Wenn die Instance im Subsystem lebt → prüft die Rollen von Instigator und Target.
* Standalone / PIE → immer `true`.

Alle eingebauten SideEffects rufen diese Funktion intern auf. Du musst sie nur dann selbst aufrufen, wenn dein eigener C++-SideEffect GAS-Änderungen vornimmt, die server-exklusiv sein sollen.

---

## TriggerCue (Utility)

```cpp
void MayDialogueGAS::TriggerCue(UAbilitySystemComponent* ASC, const FGameplayTag& CueTag,
    const FGameplayCueParameters& Parameters, uint8 CueMode)
```

Interner Helfer für `UMayDlgSideEffect_TriggerCue`. Leitet je nach `CueMode` (0 = Execute, 1 = Add, 2 = Remove) an die entsprechende ASC-Methode weiter. In eigenen Klassen kannst du ihn nutzen, musst es aber nicht — direkter ASC-Aufruf ist genauso gut.

> 📸 **Bild-Platzhalter:** `helpers-header-overview.png` — `MayDialogueGASHelpers.h` im Editor geöffnet, alle Funktionen im Überblick.
> *Setup:* Die Datei `MayDialogueGASHelpers.h` in Rider öffnen. Alle vier Inline-Funktionen im Namespace `MayDialogueGAS` sichtbar: `GetASCFromActor`, `GetTargetActor`, `GetASCFromContext`, `IsDialogueServerAuthoritative`, `TriggerCue`. Vollständige Signaturen sichtbar.

> 📸 **Bild-Platzhalter:** `helpers-blueprint-no-code.png` — Blueprint-Graph mit `HasTag`-SideEffect, kein manueller ASC-Aufruf nötig.
> *Setup:* BP-Graph zeigt einen `Event Execute Side Effect`-Node, der direkt in eine `Get Quest Subsystem`-Node führt. Kein `Get ASC`-Aufruf sichtbar — zeigt, dass Blueprint-Nutzer die Helpers nie direkt brauchen.
