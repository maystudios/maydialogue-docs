---
description: All functions of MayDialogueSubsystem with use case and example.
---

# Subsystem API

`UMayDialogueSubsystem` is the central orchestrator — a `UWorldSubsystem` of which there is exactly one instance per world. It starts dialogues, keeps the active instance alive, and broadcasts global events.

## Access

### Blueprint

```text
[Get MayDialogue Subsystem]   ← Search for "MayDialogue" in the Node palette
```

`UMayDialogueSubsystem` is `BlueprintType` — you can create a variable of type `MayDialogue Subsystem Object Reference`, store the reference in it, and use it type-safely for delegate binding.

> 📸 **Image placeholder:** `subsystem-access-bp.png` — BP graph with subsystem access and cached variable.
> *Setup:* Any Blueprint. `Event BeginPlay` → `Get MayDialogue Subsystem` → local variable `DialogueSubsystem` (type: MayDialogue Subsystem Object Reference). Below: a second section with `Is Valid` on the variable.

### C++

```cpp
UMayDialogueSubsystem* Sub = GetWorld()->GetSubsystem<UMayDialogueSubsystem>();

// Or as a static helper (requires UMayDialogueLibrary.h):
UMayDialogueSubsystem* Sub = UMayDialogueLibrary::GetDialogueSubsystem(this);
```

---

## Starting and Stopping Dialogues

| Function | Use Case |
|---|---|
| `StartDialogue(Asset, Instigator, Target)` | Start a dialogue. Returns the instance or `nullptr` on failure. |
| `CanStartDialogue(Asset, Instigator, Target)` | Check whether a start would succeed without triggering it (e.g. for interaction prompts). |
| `StopDialogue(Instance)` | Abort a specific instance. |
| `StopAllDialogues()` | Abort all active dialogues (level change, player death). |

### Example: Start dialogue and check result

```text
[Get MayDialogue Subsystem] → [Start Dialogue]
  ├─ Asset:      DA_TraderGreeting
  ├─ Instigator: Player Pawn
  └─ Target:     Trader Actor
       │ Return Value (UMayDialogueInstance)
       ▼
[Is Valid]
  ├─ True  → All good
  └─ False → Log Warning
```

```cpp
if (auto* Inst = Sub->StartDialogue(DA_TraderGreeting, Player, Trader))
{
    // Inst is the running instance
}
```

### Example: Show interaction prompt only when dialogue is possible

```cpp
bool bCanTalk = Sub->CanStartDialogue(DA_TraderGreeting, Player, Trader);
InteractionPromptWidget->SetVisibility(bCanTalk ? ESlateVisibility::Visible : ESlateVisibility::Hidden);
```

---

## Querying the Active Dialogue

| Function | Return | Use Case |
|---|---|---|
| `GetActiveDialogue()` | `UMayDialogueInstance*` or `nullptr` | Get the instance to read variables or bind delegates directly. |
| `IsAnyDialogueActive()` | `bool` | Quick check: is anything running? |

### Example: Overlap trigger only fires when no dialogue is running

```text
[Event Begin Overlap]
    │
    ▼
[Is Any Dialogue Active]  (Get MayDialogue Subsystem → Is Any Dialogue Active)
    │ (bool)
    ▼
[Branch]
  ├─ False → [Start Dialogue] ...
  └─ True  → (do nothing)
```

```cpp
void ATrigger::OnBeginOverlap(AActor* Other)
{
    if (!Sub->IsAnyDialogueActive())
    {
        Sub->StartDialogue(DA_Hint, Other, this);
    }
}
```

---

## Global Event Delegates

The subsystem delegates fire **for every dialogue** — ideal for global listeners (audio ducking, analytics, quest log).

| Delegate | When | Parameters |
|---|---|---|
| `OnAnyDialogueStarted` | Directly after a new instance starts | `UMayDialogueInstance*` |
| `OnAnyDialogueEnded` | Directly after an instance ends (regardless of Completed/Aborted) | `UMayDialogueInstance*` |
| `OnAnyDialogueAborted` | When an instance is aborted — fires **before** `OnAnyDialogueEnded` | `UMayDialogueAsset*, EMayDialogueExitStatus, float, AActor*, AActor*` |

### `FireDialogueEvent` — Trigger an event by tag

Fires a named dialogue event in the currently running instance (equivalent to a FireEvent Node in the graph, but from external code):

```text
[Get MayDialogue Subsystem]
    │
    ▼
[Fire Dialogue Event]
  ├─ World Context: Self
  └─ Event Tag:     Dialogue.Event.LightsOut
```

```cpp
UMayDialogueLibrary::FireDialogueEvent(this, TAG_Dialogue_Event_LightsOut);
// or directly on the subsystem:
Sub->FireDialogueEvent(this, TAG_Dialogue_Event_LightsOut);
```

### Participant Lookups

| Function | Type | Return |
|---|---|---|
| `Get All Participants` | Pure | `TArray<UMayDialogueParticipant*>` of all active Participants in the world |
| `Find Participant By Tag` | Pure | `UMayDialogueParticipant*` for a specific `FGameplayTag`, or `nullptr` |

Both are available in `UMayDialogueLibrary` as `BlueprintPure` (no subsystem access needed).

---

### Blueprint: Binding to an event

> 📸 **Image placeholder:** `subsystem-delegate-bind-bp.png` — BP graph for event binding on the subsystem.
> *Setup:* Quest Director Blueprint, `Event BeginPlay`. `Get MayDialogue Subsystem` → `Bind Event to On Any Dialogue Ended` → Custom Event `Handle Dialogue Ended` (Input pin: `Instance` of type `MayDialogueInstance Object Reference`). Custom Event has a `Branch` Node below: `Get Dialogue Asset` from the instance compared with an asset reference.

```text
[Event BeginPlay]
    │
    ▼
[Get MayDialogue Subsystem]
    │
    ├─→ [Bind Event to On Any Dialogue Started]  ──► Custom Event: On Dialogue Started
    └─→ [Bind Event to On Any Dialogue Ended]    ──► Custom Event: On Dialogue Ended
```

### C++: Audio ducking example

```cpp
void UAudioManager::BeginPlay()
{
    if (auto* Sub = GetWorld()->GetSubsystem<UMayDialogueSubsystem>())
    {
        Sub->OnAnyDialogueStarted.AddDynamic(this, &UAudioManager::DuckMusic);
        Sub->OnAnyDialogueEnded.AddDynamic(this, &UAudioManager::RestoreMusic);
    }
}

void UAudioManager::DuckMusic(UMayDialogueInstance* Instance)
{
    MusicComponent->SetVolumeMultiplier(0.2f);
}

void UAudioManager::RestoreMusic(UMayDialogueInstance* Instance)
{
    MusicComponent->SetVolumeMultiplier(1.0f);
}
```

---

## Read/Write Directly on the Subsystem

The subsystem exposes all read and write methods of the active instance as Blueprint-callable — you do not need to hold an instance reference.

Full description: [Read/Write API](read-write-api.md).
