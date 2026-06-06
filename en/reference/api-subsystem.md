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
// C++ — three equivalent paths
UMayDialogueSubsystem* Sub = GetWorld()->GetSubsystem<UMayDialogueSubsystem>();

// Static convenience accessor (1.0)
UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(this);

// Via Library helper
UMayDialogueSubsystem* Sub = UMayDialogueLibrary::GetDialogueSubsystem(this);
```

```text
// Blueprint
[Get MayDialogue Subsystem]    ← Library node, or
[UMayDialogueSubsystem::Get]   ← direct static node (DisplayName: "Get MayDialogue Subsystem")
```

### UMayDialogueSubsystem::Get (1.0)

```cpp
UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem",
    meta = (WorldContext = "WorldContext", DisplayName = "Get MayDialogue Subsystem"))
static UMayDialogueSubsystem* Get(const UObject* WorldContext);
```

Null-safe static accessor. Returns `nullptr` when `WorldContext` is null, has no world, or the subsystem is not present. Equivalent to `UMayDialogueLibrary::GetDialogueSubsystem(WorldContext)`.

```cpp
// Null-safe pattern
if (auto* S = UMayDialogueSubsystem::Get(this))
{
    S->StartDialogue(Asset, Instigator, Target);
}
```

> 📸 **Image placeholder:** `subsystem-access-bp.png` — Blueprint node `Get MayDialogue Subsystem` with a return pin.
> *Setup:* Any Blueprint, right-click → type "Get MayDialogue Subsystem". Node with a yellow Subsystem return pin visible. Return pin goes into a variable.

---

## Dialogue Lifecycle Methods

| Signature | Return | Description |
|---|---|---|
| `StartDialogue(Asset, Instigator, Target)` | `UMayDialogueInstance*` | Pre-flight check → create Instance → start entry. `nullptr` on error. Server-only. |
| `K2_CanStartDialogue(Asset, Instigator, Target)` | `bool` | Blueprint pre-flight check. Returns `true` if `StartDialogue` would succeed. |
| `K2_AbortDialogue(Instance)` | `void` | Abort a specific Instance. No-op on `nullptr`. Server/authority only. |
| `AbortAllDialogues()` | `void` | Abort all active Instances. Server/authority only. |
| `StopDialogue(Instance)` | `void` | **Deprecated** — use `K2_AbortDialogue`. |
| `StopAllDialogues()` | `void` | **Deprecated** — use `AbortAllDialogues`. |

```cpp
UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem")
UMayDialogueInstance* StartDialogue(UMayDialogueAsset* Asset, AActor* Instigator, AActor* Target);

// Blueprint: "Can Start Dialogue" — pre-flight validation (1.0)
UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem",
    DisplayName = "Can Start Dialogue")
bool K2_CanStartDialogue(UMayDialogueAsset* Asset, AActor* Instigator, AActor* Target);

// Blueprint: "Abort Dialogue" (1.0, canonical verb)
UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem",
    meta = (DisplayName = "Abort Dialogue"))
void K2_AbortDialogue(UMayDialogueInstance* Instance);

UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem")
void AbortAllDialogues();

// Deprecated aliases — kept for source compatibility, will be removed in a future version
UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem",
    meta = (DeprecatedFunction,
        DeprecationMessage = "Use K2_AbortDialogue instead."))
void StopDialogue(UMayDialogueInstance* Instance);

UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem",
    meta = (DeprecatedFunction,
        DeprecationMessage = "Use AbortAllDialogues instead."))
void StopAllDialogues();
```

### Verb Convention (1.0)

MayDialogue uses one canonical verb for "end a dialogue early" at every public layer:

| Layer | Canonical verb |
|---|---|
| `UMayDialogueSubsystem` | `K2_AbortDialogue` / `AbortAllDialogues` |
| `UMayDialogueLibrary` | `AbortDialogue` / `AbortAllDialogues` |
| `UMayDialogueInstance` | `AbortDialogue` (server-authoritative) |
| `IMayDialogueBridge` | `AbortDialogue` |
| `UMayDialogueParticipant` (client input) | `RequestAbortDialogue` → `ServerAbortConversation` |

- **`Abort*`** — call from server/authority code to end a dialogue.
- **`Request*`** — net-safe wrappers on `UMayDialogueParticipant`; route through Server RPCs so they work identically on clients and listen-server hosts. Designers wire UI buttons directly to `Request*`.
- **`Server*` / `Client*`** — raw UE RPC layer; do not call from gameplay code, use `Request*` instead.
- **`Stop*`** — deprecated aliases; compile with a deprecation warning and forward to `Abort*`.

**StartDialogue flow:**

```text
StartDialogue called
    │
    ▼
CanStartDialogue?  →  No → nullptr
    │ Yes
    ▼
Dialogue active?  →  Yes → AbortAllDialogues
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

## K2_CanStartDialogue — Pre-flight Check (1.0)

Use before showing interaction prompts or spending resources on a dialogue start:

```text
[Get MayDialogue Subsystem] → [Can Start Dialogue]
  ├─ Asset:      DA_Greeting
  ├─ Instigator: Player Pawn
  └─ Target:     Guard Actor
       │ (bool)
       ▼
[Branch]
  ├─ True  → Show interaction prompt
  └─ False → Hide prompt (no asset / no entry / no participant)
```

Checks: asset is valid and compiled, entry point exists, at least one of Instigator/Target carries a `UMayDialogueParticipant`. Does **not** start the dialogue. Safe to call on server or client.

---

## Safety Contracts

- All methods are **Game Thread only**.
- `StartDialogue` with a `nullptr` asset logs a warning and returns `nullptr` — no crash.
- Multiple aborts of the same Instance are idempotent.
- During `AbortAllDialogues`, calling `StartDialogue` from within delegate handlers is **not allowed** (re-entrant guard; a warning is logged and the call is discarded).
- `StartDialogue` must be called server-side — clients route through `UMayDialogueParticipant::RequestStartDialogue`.

---

## Deinitialization (Level Travel)

UE calls `Deinitialize()` on world change:

1. `AbortAllDialogues()` — cleanly abort all Instances.
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
