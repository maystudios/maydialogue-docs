---
description: UMayDialogueLibrary — static Blueprint helpers for a quick start.
---

# Blueprint Library

`UMayDialogueLibrary` is a Blueprint Function Library with static methods. It is the simplest way to control MayDialogue from any Blueprint — without caching a subsystem reference first.

{% hint style="info" %}
The Library is a pure convenience layer. It has no state of its own and delegates everything to the subsystem. If you want to bind delegates, you must address the subsystem directly — the Library has no events of its own.
{% endhint %}

---

## Methods Overview

### Base Library (`UMayDialogueLibrary`)

| Function | Type | When to use |
|---|---|---|
| `Start Dialogue` | Callable | Start a dialogue with a specific asset. Returns Instance or `nullptr`. |
| `Stop Dialogue` | Callable | Abort a specific instance. |
| `Stop All Dialogues` | Callable | Abort all active dialogues (player death, pause, level change). |
| `Get Active Dialogue` | Pure | Read the active instance (e.g. to query variables). |
| `Is Any Dialogue Active` | Pure | Check if a dialogue is currently running. |
| `Get Dialogue Subsystem` | Callable | Get the subsystem reference (e.g. for delegate binding). |
| `Get All Participants` | Pure | All active `UMayDialogueParticipant`s in the world. |
| `Find Participant By Tag` | Pure | Find a Participant by `FGameplayTag`. |

#### TaskResult Make-Helpers (Category `MayDialogue|Task Result`)

For the return value of `ExecuteNode` overrides in custom Node Blueprints:

| Make Node | Meaning |
|---|---|
| `Make Advance` | Normal advance (with optional NextNodeGuid override) |
| `Make Abort` | Abort dialogue |
| `Make Wait` | Pause dialogue (async Nodes) |
| `Make Pause And Present Choices` | Open Choice screen |
| `Make Advance With Choice` | Choice + immediate advance |
| `Make Return To Start` | Jump back to the beginning of the graph |
| `Make Return To Last` | Return to the last visited Node |
| `Make Return To Current` | Repeat the current Node |

---

### GAS Library (Category `MayDialogue|GAS`)

| Function | Type | Description |
|---|---|---|
| `Is Dialogue Server Authoritative` | Pure | Whether the current dialogue context is server-authoritative |
| `Get ASC From Context` | Pure | `UAbilitySystemComponent*` from `FMayDialogueContext` — no manual cast |
| `Trigger Cue On Context` | Callable | Fire a Gameplay Cue directly from a context |

---

### Async Library (Category `MayDialogue|Async`)

| Function | Type | Description |
|---|---|---|
| `Request Node Advance` | Callable | Trigger async Node transition — BP wrapper around `ForceTransitionToNode` with null guard |
| `Evaluate All Requirements` | Pure | Evaluate all Requirements in an array and return the aggregated result |
| `Execute All Side Effects` | Callable | Execute server-side SideEffects of an array |
| `Execute All Client Side Effects` | Callable | Execute client-side (cosmetic) SideEffects of an array |

---

### Variables Library (Category `MayDialogue|Variables`)

| Function | Type | Description |
|---|---|---|
| `Get Dialogue Variable` | Callable | Read a variable from an `FMayDialogueVariables` container |
| `Set Dialogue Variable` | Callable | Write a variable to an `FMayDialogueVariables` container |
| `Copy Dialogue Variables` | Callable | **Planned** — not yet available. |

---

## When to use the Library vs. the Subsystem Directly?

| Scenario | Recommendation |
|---|---|
| Quick start call from a Widget or LevelScript | Library |
| You need the subsystem anyway (event binding) | Subsystem directly and cache the reference |
| Repeated calls in the same scope | Get subsystem reference once and cache it |

---

## Recipes

### Start a Dialogue

> 📸 **Image placeholder:** `library-start-dialogue-bp.png` — BP graph: Start Dialogue Library call with error Branch.
> *Setup:* Interaction Component Blueprint or LevelScript. `Event On Interact` → `Start Dialogue` (Library, category MayDialogue). Pins: `Asset` = dialogue asset reference, `Instigator` = Player Pawn reference, `Target` = Self. Return Value pin → `Is Valid` → Branch: True = (continue), False = `Print String "Dialog failed"`.

```text
[Start Dialogue]  (Category: MayDialogue)
  ├─ Asset:      DA_VillagerIntro
  ├─ Instigator: Player Pawn
  └─ Target:     Self (NPC)
       │ Return Value
       ▼
[Is Valid]
  ├─ True  → Dialogue is running
  └─ False → Error fallback
```

---

### Reading the Running Dialogue

> 📸 **Image placeholder:** `library-get-active-bp.png` — BP graph: Is Any Dialogue Active → Branch → Get Active Dialogue.
> *Setup:* HUD Blueprint, `Event Tick` or button press event. `Is Any Dialogue Active` → Branch. True branch: `Get Active Dialogue` → (process UMayDialogueInstance reference, e.g. `Get Status`).

```text
[Is Any Dialogue Active]
    │ (bool)
    ▼
[Branch]
  └─ True → [Get Active Dialogue] → Use instance
```

---

### Abort All Dialogues on Player Death

> 📸 **Image placeholder:** `library-stop-all-bp.png` — BP graph: On Player Died → Stop All Dialogues.
> *Setup:* Character Blueprint or GameMode. `Event On Player Died` (Custom Event) → `Stop All Dialogues` (Library Node). No further connections needed.

```text
[Event On Player Died]
    │
    ▼
[Stop All Dialogues]
```

---

### Get Subsystem for Delegate Binding

```text
[Get Dialogue Subsystem]
    │ (store subsystem reference in variable)
    ▼
[Bind Event to On Any Dialogue Started]  ──► Custom Event: Handle Dialogue Started
[Bind Event to On Any Dialogue Ended]    ──► Custom Event: Handle Dialogue Ended
```

---

## C++ Usage

```cpp
// Start dialogue
UMayDialogueInstance* Inst = UMayDialogueLibrary::StartDialogue(
    this, DA_VillagerIntro, Player, NPC);

// Check if dialogue is running
if (UMayDialogueLibrary::IsAnyDialogueActive(this))
{
    UMayDialogueLibrary::StopAllDialogues(this);
}

// Get subsystem
UMayDialogueSubsystem* Sub = UMayDialogueLibrary::GetDialogueSubsystem(this);
```

{% hint style="info" %}
In C++, direct subsystem access is more idiomatic than the Library. The Library is primarily intended for Blueprint users.
{% endhint %}
