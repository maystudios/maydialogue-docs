---
description: Three methods for starting a dialogue — with Blueprint and C++.
---

# Starting a Dialogue

All three approaches lead to the same instance. Pick the one that fits your code best.

---

## Option 1 — Participant Component

**When:** The NPC has a `MayDialogueParticipant` component and should start its `DefaultDialogue` when the player interacts.

### Blueprint

> 📸 **Image placeholder:** `start-participant-bp.png` — BP graph of the NPC interaction event.
> *Setup:* Open NPC Blueprint. Graph shows: `Event On Interact` (Custom Event, one `AActor` pin `Instigator`) → `Get Component by Class` (Class: `MayDialogueParticipant`, Target: Self) → `Start Default Dialogue` (Instigator pin: `Instigator` variable). Return pin (UMayDialogueInstance) left unconnected. All Nodes cleanly labeled.

```text
[Event On Interact (Instigator: Actor)]
    │
    ▼
[Get Component by Class: MayDialogueParticipant] (Target: Self)
    │
    ▼
[Start Default Dialogue]
    └─ Instigator: Instigator pin from On Interact
```

### C++

```cpp
void AGuardActor::OnInteract(AActor* Instigator)
{
    if (auto* Part = FindComponentByClass<UMayDialogueParticipant>())
    {
        Part->StartDefaultDialogue(Instigator);
    }
}
```

{% hint style="info" %}
`StartDefaultDialogue` starts the asset registered in the component's `DefaultDialogue` field. If you want to play a different asset, call `SetActiveDialogue(MyAsset)` first.
{% endhint %}

---

## Option 2 — Library (recommended for Blueprints)

**When:** You want to start a dialogue with a specific asset without needing references to Participant components first. Works from any Blueprint: Widget, GameMode, LevelScript.

### Blueprint

> 📸 **Image placeholder:** `start-library-bp.png` — BP graph with Library Node.
> *Setup:* Any Blueprint (e.g. LevelBlueprint). Graph: `Event BeginPlay` → `Start Dialogue` (Library Node from category MayDialogue). Pins filled in: `Asset` = hardcoded dialogue asset reference, `Instigator` = `Get Player Pawn (0)`, `Target` = NPC actor reference from scene. Return Value pin goes into an `Is Valid` Branch.

```text
[Start Dialogue]  (Category: MayDialogue)
  ├─ Asset:      DA_Greeting
  ├─ Instigator: Get Player Pawn (0)
  └─ Target:     Guard Actor Reference
       │
       ▼
[Is Valid: Return Value]
  ├─ True  → Dialogue is running
  └─ False → Print String: "Dialogue could not start"
```

### C++

```cpp
UMayDialogueInstance* Inst = UMayDialogueLibrary::StartDialogue(
    this,           // WorldContext (any UObject in the world)
    DA_Greeting,    // UMayDialogueAsset*
    PlayerPawn,     // Instigator
    GuardActor      // Target
);
```

---

## Option 3 — Subsystem Directly

**When:** Your system code (Quest Director, Cutscene Manager, Tutorial Script) already references the subsystem and also needs delegates.

### Blueprint

> 📸 **Image placeholder:** `start-subsystem-bp.png` — BP graph with subsystem access.
> *Setup:* Blueprint with `Get MayDialogue Subsystem` → `Start Dialogue` (on the Subsystem Node). `Instigator` = Player Pawn, `Target` = NPC reference. Subsystem reference cached in a local variable, then directly below `Bind Event to On Any Dialogue Ended` connected to a Custom Event.

```text
[Get MayDialogue Subsystem]
    │
    ├─→ [Start Dialogue]
    │       ├─ Asset:      DA_IntroScene
    │       ├─ Instigator: Player Pawn
    │       └─ Target:     NPC Ref
    │
    └─→ [Bind Event to On Any Dialogue Ended]
            └─ Event: Handle Dialogue Ended (Custom Event)
```

### C++

```cpp
if (auto* Sub = GetWorld()->GetSubsystem<UMayDialogueSubsystem>())
{
    Sub->OnAnyDialogueEnded.AddDynamic(this, &AQuestDirector::HandleDialogueEnded);
    Sub->StartDialogue(DA_IntroScene, PlayerPawn, NPC);
}
```

---

## Check First: CanStartDialogue

If you want to know whether a dialogue can start without starting it:

```text
[Get MayDialogue Subsystem] → [Can Start Dialogue]
  ├─ Asset:      DA_Greeting
  ├─ Instigator: Player Pawn
  └─ Target:     Guard Actor
       │ (bool)
       ▼
[Branch]
  ├─ True  → Show interaction prompt
  └─ False → Hide prompt
```

```cpp
bool bOk = Sub->CanStartDialogue(DA_Greeting, Player, Guard);
```

Checks: asset valid, Entry present, at least one Participant with matching tags.

---

## Stopping a Running Dialogue

```text
[Get MayDialogue Subsystem] → [Stop All Dialogues]
```

```cpp
Sub->StopAllDialogues();   // stop all (level change, player death)
Sub->StopDialogue(Inst);   // stop a specific instance
```

> 📸 **Image placeholder:** `stop-all-dialogues-bp.png` — BP graph for player death handling.
> *Setup:* Game Mode or Character Blueprint. `Event On Player Died` → `Stop All Dialogues` (Library Node). No further output.

{% hint style="warning" %}
If you call `StartDialogue` while a dialogue is already running, the old one is **immediately aborted**. No queue, no crash.
{% endhint %}
