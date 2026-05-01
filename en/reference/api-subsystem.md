---
description: All public methods and delegates of UMayDialogueSubsystem.
---

# API: UMayDialogueSubsystem

Central orchestrator. A `UWorldSubsystem` — exactly one instance per world. Also implements `FTickableGameObject` (for auto-advance/watchdog) and `IMayDialogueBridge` (for external consumers).

- **Blueprint access**: In the Blueprint editor under category *MayDialogue|Subsystem*.
- **Module**: `MayDialogue`
- **Base**: `UWorldSubsystem`

---

## Access

```cpp
// C++
UMayDialogueSubsystem* Sub = GetWorld()->GetSubsystem<UMayDialogueSubsystem>();

// Via Library helper
UMayDialogueSubsystem* Sub = UMayDialogueLibrary::GetDialogueSubsystem(this);
```

```text
// Blueprint
[Get MayDialogue Subsystem]
```

> 📸 **Image placeholder:** `subsystem-access-bp.png` — Blueprint node `Get MayDialogue Subsystem` with a return pin.
> *Setup:* Any Blueprint, right-click → type "Get MayDialogue Subsystem". Node with a yellow Subsystem return pin visible. Return pin goes into a variable.

---

## Dialogue Lifecycle Methods

| Signature | Return | Description |
|---|---|---|
| `StartDialogue(Asset, Instigator, Target)` | `UMayDialogueInstance*` | Pre-flight check → create Instance → start entry. `nullptr` on error. |
| `CanStartDialogue(Asset, Instigator, Target)` | `bool` | Pure query. Checks whether `StartDialogue` would succeed. |
| `StopDialogue(Instance)` | `void` | Aborts a specific Instance. No-op on `nullptr`. |
| `StopAllDialogues()` | `void` | Aborts all active Instances. |

```cpp
UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem")
UMayDialogueInstance* StartDialogue(UMayDialogueAsset* Asset, AActor* Instigator, AActor* Target);

UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem")
bool CanStartDialogue(UMayDialogueAsset* Asset, AActor* Instigator, AActor* Target) const;

UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem")
void StopDialogue(UMayDialogueInstance* Instance);

UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem")
void StopAllDialogues();
```

**StartDialogue flow:**

```text
StartDialogue called
    │
    ▼
CanStartDialogue?  →  No → nullptr
    │ Yes
    ▼
Dialogue active?  →  Yes → StopAllDialogues
    │
    ▼
Create new Instance
    │
    ▼
Resolve participants
    │
    ▼
Broadcast OnAnyDialogueStarted
    │
    ▼
Start entry node → return Instance
```

---

## Query Methods

| Signature | Return | Description |
|---|---|---|
| `GetActiveDialogue()` | `UMayDialogueInstance*` | Active Instance or `nullptr`. |
| `IsAnyDialogueActive()` | `bool` | Checks whether anything is running. |

```cpp
UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem")
UMayDialogueInstance* GetActiveDialogue() const;

UFUNCTION(BlueprintPure, Category = "MayDialogue|Subsystem")
bool IsAnyDialogueActive() const;
```

---

## Subsystem Delegates

| Delegate | Type | Fire Time |
|---|---|---|
| `OnAnyDialogueStarted` | `FOnAnyDialogueEvent` | Directly after an Instance starts. |
| `OnAnyDialogueEnded` | `FOnAnyDialogueEvent` | Directly after an Instance ends. |
| `OnAnyDialogueAborted` | `FOnMayDialogueAborted` | Any dialogue is aborted — fires before `OnAnyDialogueEnded`. |

```cpp
UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnAnyDialogueEvent OnAnyDialogueStarted;

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnAnyDialogueEvent OnAnyDialogueEnded;

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnMayDialogueAborted OnAnyDialogueAborted;
```

`FOnAnyDialogueEvent` = `DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnAnyDialogueEvent, UMayDialogueInstance*, Instance)`

---

## Read/Write Methods (Bridge API)

All directly on the Subsystem as Blueprint-Callable — operate on the active Instance.

| Blueprint Name | Return | Description |
|---|---|---|
| `Get Active Dialogue Asset` | `UMayDialogueAsset*` | Asset of the active Instance. |
| `Get Current Node GUID` | `FGuid` | GUID of the running node. |
| `Get Active Participants` | `TArray<AActor*>` | All participating actors. |
| `Get Dialogue Variable (As String)` | `bool` | Reads a dialogue variable as a string. Out param: value. |
| `Get Participant Variable (As String)` | `bool` | Reads a participant variable as a string. |
| `Get Pending Choices` | `TArray<FMayDialogueChoiceEntry>` | Current choice list (empty if no PlayerChoice is active). |
| `Set Dialogue Variable (From String)` | `bool` | Sets a dialogue variable from a string. |
| `Set Participant Variable (From String)` | `bool` | Sets a participant variable from a string. |
| `Select Choice` | `bool` | Selects a choice by index. |
| `Force Advance` | `bool` | Skips the current advance wait. |

Details with examples: [Read/Write API](../runtime/read-write-api.md).

---

## Safety Contracts

- All methods are **Game Thread only**.
- `StartDialogue` with a `nullptr` asset logs a warning and returns `nullptr` — no crash.
- Multiple aborts of the same Instance are idempotent.
- During `StopAllDialogues`, calling `StartDialogue` from within delegate handlers is **not allowed** (re-entrant guard; a warning is logged and the call is discarded).

---

## Deinitialization (Level Travel)

UE calls `Deinitialize()` on world change:

1. `StopAllDialogues()` — cleanly abort all Instances.
2. Detach timers and delegates.
3. Final `CleanupCompletedDialogues()` flush.

The new level gets a fresh Subsystem object.

---

## Replication

The Subsystem is **not replicated**. Each client has its own local Subsystem object. Multiplayer sync runs via `UMayDialogueParticipant` RPCs.

## See Also

- [API: Library](api-library.md) — convenience wrappers.
- [API: Delegates](api-delegates.md) — complete delegate signatures.
- [Runtime → Subsystem API](../runtime/subsystem-api.md) — guided walkthrough with examples.
