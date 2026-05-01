---
description: All methods of UMayDialogueLibrary — signatures, parameters, return values, purpose.
---

# API: UMayDialogueLibrary

Blueprint Function Library with static methods. Convenience layer over the Subsystem — no own state, no own delegates.

- **Blueprint category**: In the Blueprint editor under category *MayDialogue*.
- **Module**: `MayDialogue`

> 📸 **Image placeholder:** `library-node-palette.png` — Blueprint node palette with all Library nodes.
> *Setup:* Open any Blueprint, right-click in the graph, enter "MayDialogue" in the search. Screenshot of the palette with all visible nodes: Start Dialogue, Stop Dialogue, Stop All Dialogues, Get Active Dialogue, Is Any Dialogue Active, Get Dialogue Subsystem. Category header "MayDialogue" visible.

---

## Method Overview

| Function | Kind | Parameters | Return | When to use |
|---|---|---|---|---|
| `Start Dialogue` | Callable | `WorldContext`, `Asset`, `Instigator`, `Target` | `UMayDialogueInstance*` | Start a dialogue from any Blueprint. |
| `Stop Dialogue` | Callable | `Instance` | — | Stop a specific Instance. |
| `Stop All Dialogues` | Callable | `WorldContext` | — | Abort all dialogues (death, level travel). |
| `Get Active Dialogue` | Callable | `WorldContext` | `UMayDialogueInstance*` | Fetch the active Instance to read state. |
| `Is Any Dialogue Active` | Pure | `WorldContext` | `bool` | Check whether a dialogue is running (e.g. for overlap guards). |
| `Get Dialogue Subsystem` | Callable | `WorldContext` | `UMayDialogueSubsystem*` | Fetch the Subsystem for delegate binding. |

`WorldContext` is automatically filled in by the node context in Blueprint (no manual pin needed).

---

## Signatures

### Start Dialogue

```cpp
UFUNCTION(BlueprintCallable, Category = "MayDialogue",
    meta = (WorldContext = "WorldContext", DefaultToSelf = "Instigator"))
static UMayDialogueInstance* StartDialogue(
    UObject*           WorldContext,
    UMayDialogueAsset* Asset,
    AActor*            Instigator,
    AActor*            Target);
```

Returns the created Instance or `nullptr` on error (no asset, no entry, Instigator/Target invalid). Restarting a running dialogue aborts the old one.

---

### Stop Dialogue

```cpp
UFUNCTION(BlueprintCallable, Category = "MayDialogue")
static void StopDialogue(UMayDialogueInstance* Instance);
```

No-op on `nullptr`. Cleanly aborts the specified Instance.

---

### Stop All Dialogues

```cpp
UFUNCTION(BlueprintCallable, Category = "MayDialogue",
    meta = (WorldContext = "WorldContext"))
static void StopAllDialogues(UObject* WorldContext);
```

Aborts all active Instances. Typical for level travel, player death, pause menu.

---

### Get Active Dialogue

```cpp
UFUNCTION(BlueprintCallable, Category = "MayDialogue",
    meta = (WorldContext = "WorldContext"))
static UMayDialogueInstance* GetActiveDialogue(UObject* WorldContext);
```

Returns the (only) active Instance or `nullptr`. Since the Subsystem allows only one dialogue at a time, this is practically always unambiguous.

---

### Is Any Dialogue Active

```cpp
UFUNCTION(BlueprintPure, Category = "MayDialogue",
    meta = (WorldContext = "WorldContext"))
static bool IsAnyDialogueActive(UObject* WorldContext);
```

Quick query. Cost-free — internally just looks up `Subsystem->IsAnyDialogueActive()`.

---

### Get Dialogue Subsystem

```cpp
UFUNCTION(BlueprintCallable, Category = "MayDialogue",
    meta = (WorldContext = "WorldContext"))
static UMayDialogueSubsystem* GetDialogueSubsystem(UObject* WorldContext);
```

Returns the Subsystem. If you need it frequently, cache it in a Blueprint variable.

---

## Blueprint Examples

### Start with Error Check

```text
[Start Dialogue]
  ├─ Asset:      DA_Merchant
  ├─ Instigator: Player Pawn
  └─ Target:     Merchant Actor
       │ Return Value
       ▼
[Is Valid]  → Branch
  ├─ True  → Continue (dialogue running)
  └─ False → Print String "Dialogue failed"
```

> 📸 **Image placeholder:** `library-start-with-check-bp.png` — Complete Blueprint graph: Start Dialogue → Is Valid → Branch.
> *Setup:* Interaction Blueprint. Nodes: `Start Dialogue` (pins filled), `Is Valid` (Input: Return Value of the Start node), `Branch`, True branch empty, False branch → `Print String`. All connections visible.

### Stop All Dialogues on Player Death

```text
[Event On Player Died]
    │
    ▼
[Stop All Dialogues]
```

### Get Subsystem for Delegate Binding

```text
[Get Dialogue Subsystem] → (Store in variable "Subsystem")
    │
    ▼
[Bind Event to On Any Dialogue Ended]  ──► Custom Event: Handle Ended
```

---

## What the Library Can't Do

The Library has **no delegate properties**. For `OnAnyDialogueStarted` / `OnAnyDialogueEnded` you need the Subsystem directly (via `Get Dialogue Subsystem`).

Also not in the Library:
- Variable Get/Set → on the Subsystem or the Instance directly
- `CanStartDialogue` → on the Subsystem directly

## See Also

- [API: Subsystem](api-subsystem.md) — complete Subsystem API with delegates.
- [API: Delegates](api-delegates.md) — all delegate signatures.
- [Runtime → Starting a Dialogue](../runtime/starting-dialogues.md) — guided walkthrough.
