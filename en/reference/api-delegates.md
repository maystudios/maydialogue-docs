---
description: All delegate types with signature, fire time, and typical use cases.
---

# API: Delegates & Events

MayDialogue broadcasts on two levels: **Instance** (per conversation) and **Subsystem** (global). All delegates are `DYNAMIC_MULTICAST_DELEGATE` — any number of listeners, bindable from Blueprint.

---

## Overview

| Delegate | Level | Fire Time |
|---|---|---|
| `OnDialogueStarted` | Instance | Directly after Instance creation, before the first node. |
| `OnDialogueEnded` | Instance | On Exit (Completed/Failed) or Abort. |
| `OnNodeReached` | Instance | After successful node execution. |
| `OnMessageReceived` | Instance | When a SayLine assembles a message. |
| `OnChoicesPresented` | Instance | When a PlayerChoice node filters and presents options. |
| `OnChoiceMade` | Instance | After `SelectChoice(Index)`, before the transition. |
| `OnVariableChanged` | Instance | After a variable mutation (node or external code). |
| `OnDialogueEventFired` | Instance | On a FireEvent node. |
| `OnAnyDialogueStarted` | Subsystem | Every time any dialogue starts. |
| `OnAnyDialogueEnded` | Subsystem | Every time any dialogue ends. |

---

## Instance Delegates

### OnDialogueStarted

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_FourParams(
    FOnMayDialogueStarted,
    UMayDialogueAsset*, Asset,
    AActor*,            Instigator,
    AActor*,            Target,
    float,              StartTime);

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnMayDialogueStarted OnDialogueStarted;
```

**Fire time**: After Instance creation, before the first node.
**Use case**: Freeze player movement, create a quest log entry, switch music.

---

### OnDialogueEnded

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_FiveParams(
    FOnMayDialogueEnded,
    UMayDialogueAsset*,     Asset,
    EMayDialogueExitStatus, ExitStatus,
    float,                  Duration,
    AActor*,                Instigator,
    AActor*,                Target);

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnMayDialogueEnded OnDialogueEnded;
```

**Fire time**: `EndDialogue()` (Completed/Failed) or `AbortDialogue()`.
**Use case**: Release player movement, check quest completion, queue a follow-up dialogue.

---

### OnNodeReached

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnMayDialogueNodeReached,
    FGuid,                 NodeGuid,
    UMayDialogueNode_Base*, Node);

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnMayDialogueNodeReached OnNodeReached;
```

**Fire time**: Directly after `ExecuteNode(...)`.
**Use case**: Debug HUD, telemetry, breakpoint system.

---

### OnMessageReceived

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnMayDialogueMessageReceived,
    const FMayDialogueMessage&, Message);

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnMayDialogueMessageReceived OnMessageReceived;
```

**Fire time**: Inside `SayLine::ExecuteNode`, after `FMayDialogueMessage` has been assembled.
**Use case**: UI widget renders speaker name, text, and portrait.

---

### OnChoicesPresented

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnMayDialogueChoicesPresented,
    const TArray<FMayDialogueChoiceEntry>&, Choices);

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnMayDialogueChoicesPresented OnChoicesPresented;
```

**Fire time**: `Instance::PresentChoices(...)`.
**Use case**: Render choice buttons, start timeout bar, controller vibration.

---

### OnChoiceMade

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnMayDialogueChoiceMade,
    int32,                         ChoiceIndex,
    const FGameplayTagContainer&,  ChoiceTags);

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnMayDialogueChoiceMade OnChoiceMade;
```

**Fire time**: `Instance::SelectChoice(Index)`, before the transition.
**Use case**: Analytics ("which choices do players pick?"), achievement trigger.

{% hint style="info" %}
`ChoiceTags` is the `FGameplayTagContainer` from the chosen `FMayDialogueChoiceEntry`. Quest systems and analytics can react to tag metadata directly from the delegate without re-querying the instance.
{% endhint %}

---

### OnVariableChanged

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_FiveParams(
    FOnMayDialogueVariableChanged,
    FName,                     VariableName,
    EMayDialogueVariableScope, Scope,
    EMayDialogueVariableType,  Type,
    const FString&,            OldValueAsString,
    const FString&,            NewValueAsString);

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnMayDialogueVariableChanged OnVariableChanged;
```

**Fire time**: After a SetVariable node or an external `SetDialogueVariable` call.
**Use case**: HUD update (disposition display), live quest status, undo preview.

{% hint style="info" %}
Both the old and new values are provided as strings. Use `Type` to parse them back (e.g. `FCString::Atoi` for Int). `OldValueAsString` is empty when the variable did not exist before this mutation.
{% endhint %}

---

### OnDialogueEventFired

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnMayDialogueEventFired,
    const FGameplayTag&, EventTag);

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnMayDialogueEventFired OnDialogueEventFired;
```

**Fire time**: `FireEvent` Action Node in the dialogue graph.
**Use case**: Loose coupling to external systems — light effects, AI switch, sound trigger.

---

## Subsystem Delegates

### OnAnyDialogueStarted / OnAnyDialogueEnded

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnAnyDialogueEvent,
    UMayDialogueInstance*, Instance);

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnAnyDialogueEvent OnAnyDialogueStarted;

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnAnyDialogueEvent OnAnyDialogueEnded;
```

**Fire time**: Subsystem level, in parallel with the Instance variant.
**Use case**: Audio ducking, global input mode switch, analytics.

---

## Binding Patterns

### Blueprint: Bind an Instance Delegate

> 📸 **Image placeholder:** `delegate-bind-instance-bp.png` — Blueprint graph: Start Dialogue → Bind Event to On Dialogue Ended.
> *Setup:* NPC Blueprint. `Start Dialogue` (Return Value = Instance) → `Bind Event to On Dialogue Ended` (Instance pin = Return Value, Event pin = Custom Event `Handle Dialogue Ended`). Custom Event has params: `Asset`, `ExitStatus`, `Duration`, `Instigator`, `Target`. Below: `Switch on EMayDialogueExitStatus`.

```text
[Start Dialogue] → Return Value (Instance)
    │
    ▼
[Bind Event to On Dialogue Ended]  ──► Custom Event: Handle Dialogue Ended
                                             (ExitStatus, Duration, ...)
```

### Blueprint: Bind a Subsystem Delegate

```text
[Event BeginPlay]
    │
    ▼
[Get Dialogue Subsystem]
    │
    ▼
[Bind Event to On Any Dialogue Started]  ──► Custom Event: Handle Any Start
[Bind Event to On Any Dialogue Ended]    ──► Custom Event: Handle Any End
```

### C++: Instance Delegate

```cpp
UMayDialogueInstance* Inst = Sub->StartDialogue(Asset, Player, NPC);
if (Inst)
{
    Inst->OnDialogueEnded.AddDynamic(this, &AMyActor::HandleEnd);
    Inst->OnVariableChanged.AddDynamic(this, &AMyActor::HandleVarChange);
}

// Handler must match the 5-param signature:
// void AMyActor::HandleVarChange(FName VarName, EMayDialogueVariableScope Scope,
//     EMayDialogueVariableType Type, const FString& OldValue, const FString& NewValue);
```

### C++: Subsystem Delegate + Cleanup

```cpp
// BeginPlay
Sub->OnAnyDialogueStarted.AddDynamic(this, &AMyActor::HandleStart);
Sub->OnAnyDialogueEnded.AddDynamic(this, &AMyActor::HandleEnd);

// EndPlay — the Subsystem lives until world end, so clean up properly
Sub->OnAnyDialogueStarted.RemoveDynamic(this, &AMyActor::HandleStart);
Sub->OnAnyDialogueEnded.RemoveDynamic(this, &AMyActor::HandleEnd);
```

---

## Delegate Cheatsheet: Which One to Use?

| Goal | Best Delegate |
|---|---|
| Show dialogue overlay | `OnDialogueStarted` (Subsystem or Instance) |
| Start typewriter | `OnMessageReceived` |
| Render choice buttons | `OnChoicesPresented` |
| Log the player's choice | `OnChoiceMade` |
| Complete a quest step | `OnDialogueEnded` (ExitStatus == Completed) |
| Update a HUD value live | `OnVariableChanged` |
| Trigger a light effect from the dialogue graph | `OnDialogueEventFired` |
| Release player movement | `OnDialogueEnded` |
| Duck audio for all dialogues | `OnAnyDialogueStarted` / `OnAnyDialogueEnded` on the Subsystem |

## See Also

- [Types & Enums](types.md) — `EMayDialogueExitStatus`, `EMayDialogueVariableType`, `FMayDialogueMessage`, etc.
- [Runtime → Bridge & Lifecycle Events](../runtime/bridge-events.md) — guided walkthrough with code examples.
