---
description: Save a running dialogue into the save game and resume it at exactly the right place.
---

# Resume-at-Node: Save & Resume Dialogues

A player is halfway through a long conversation — and quits the game. On the next launch the dialogue should continue right where it left off: the same line, the same variables, the same open choice. That is exactly what **Resume-at-Node** does.

MayDialogue serializes a running instance into a compact **snapshot** and can restore it later. You don't have to wire anything up by hand: the QuickSave helper captures active dialogues automatically. If you have your own SaveGame system, you call a single function.

{% hint style="info" %}
Resume-at-Node saves the **progress** of a conversation, not just participant memory. Participant memory (`PersistentMemory`) lives on independently of this — see [QuickSave Helper](../persistence/quicksave-helper.md) and [SaveGame Integration](../persistence/save-integration.md).
{% endhint %}

## The zero-code path: QuickSave captures dialogues

If you use the [QuickSave Helper](../persistence/quicksave-helper.md), resume is already built in. `QuickSaveToSlot` captures every currently running dialogue, and `QuickLoadFromSlot` resumes them automatically after loading.

**Saving** — when leaving the level or via a pause menu:

```text
[Event "Player saves"]
    │
    ▼
[Quick Save To Slot]   (category: MayDialogue|Persistence)
    ├─ World Context: Self
    ├─ Slot Name:     "AutoSave"
    └─ User Index:    0
         → captures PersistentMemory + every active dialogue snapshot
```

**Loading** — at level start:

```text
[Event Begin Play]
    │
    ▼
[Does Slot Exist? "AutoSave"]
    ├─ True  → [Quick Load From Slot "AutoSave"]
    │             → restores PersistentMemory
    │             → resumes each saved dialogue at its remembered node
    └─ False → (no save present — first launch)
```

That is all you need. The player lands back on the last line shown, and if they were in front of a choice when they saved, that exact choice is presented again.

{% hint style="warning" %}
**Load is a *restore*, not an *overlay* (behaviour change).** When `QuickLoadFromSlot` restores a participant's `PersistentMemory`, it now **resets the live memory to empty first** and then reproduces *exactly* the saved bag — a replace, not a union. This matters when you load an *earlier* save after later play: keys written after the save was taken (including the auto-written `MayDlg_Visit_*` progression counters, or any `SetPersistent*` from later in the session) are **dropped**, so loading an earlier save truly rolls back to that point instead of silently keeping later progression. The reset is scoped: only participants that are **present in the save** are reset — a participant with no saved entry keeps its live state untouched (QuickLoad for a tag with no saved data is a no-op). `GlobalMemory` set via the `SetGlobal*` helpers is also preserved across a save.
{% endhint %}

{% hint style="warning" %}
Resume is a **server operation**. In multiplayer projects, saving and resuming run exclusively on the server (the same way the dialogue itself lives only on the server — see [Subsystem API](subsystem-api.md)). Clients receive the resumed dialogue mirrored via the normal Client RPCs as soon as the server picks it back up.
{% endhint %}

> 📸 **Image placeholder:** `resume-quicksave-bp.png` — Blueprint graph: pause-menu button → QuickSaveToSlot, next to it BeginPlay → DoesSlotExist → QuickLoadFromSlot.
> *Setup:* Blueprint graph in the GameMode or Level Blueprint. Left area (comment "Save"): `On Save Clicked` → `Quick Save To Slot` (Slot="AutoSave"). Right area (comment "Load"): `Event Begin Play` → `Does Slot Exist` → `Branch` → True: `Quick Load From Slot`. With a dialogue running in the background (a dialogue widget visible in the viewport background) so it is clear the running dialogue is captured.

## What exactly is resumed — and what is not

Resume restores the **logical state** of the conversation, not every transient runtime resource. This is deliberate: a freshly launched game no longer has the old timers, no playing audio sources, and no open montage delegates.

| Restored | **Not** restored |
| --- | --- |
| Current node position (`CurrentNodeGuid`) | Running timers (e.g. an auto-advance countdown) |
| Dialogue-scope variables (`DialogueVariables`) | Voice / audio playback (does not resume mid-line) |
| Scope stack (sub-dialogue nesting) | Async node state (Wait, montage, timer) |
| Pending choice (re-presented) | UI animations / typewriter position |
| Participant tags of those involved | |

**Async nodes restart their node fresh.** If the player saved on a node that was waiting for something (a `Wait` node, a `PlayAnimation` with *Wait For Montage End*, a running auto-advance timer), that node is **re-entered** on resume — not frozen mid-wait. So the timer starts over, the animation plays again. For the player this feels natural: the line is shown cleanly from the start.

{% hint style="info" %}
**Rule of thumb:** Resume remembers *where* the dialogue is and *what it knows* — not *how far an animation had already played*. Plan save points so that resuming a line from the start is acceptable (in practice it almost always is).
{% endhint %}

## C++ / your own SaveGame system

If you have your own SaveGame system, you bypass the QuickSave helper and work with the snapshot directly.

### Create a snapshot

Enumerate the running instances via `Subsystem->GetAllActiveDialogues()` (multiple dialogues can run concurrently) and call `CreateSnapshot` on each. (For a single dialogue, `Subsystem->GetActiveDialogue()` returns just the most recent one.)

```cpp
#include "MayDialogueInstance.h"
#include "MayDialogueSaveGame.h"

// On save: write each active instance into a snapshot.
UMayDialogueSaveGame* Save = /* your save object */;

for (UMayDialogueInstance* Inst : Subsystem->GetAllActiveDialogues())
{
    FMayDialogueInstanceSnapshot Snapshot;
    if (Inst->CreateSnapshot(Snapshot))      // const — does not mutate the instance
    {
        Save->ActiveDialogueSnapshots.Add(Snapshot);
    }
}
```

`CreateSnapshot` is `const` — it reads the instance out without changing it. The running dialogue is not interrupted. It returns `bool`: `false` (leaving `OutSnapshot` untouched) when there is nothing meaningful to capture — the dialogue is inactive, already terminal, or has no resolvable current node/asset. Skip those instances, as the loop above does.

### Resume a snapshot

On load, hand each saved snapshot back to the subsystem:

```cpp
UMayDialogueSubsystem* Subsystem = GetWorld()->GetSubsystem<UMayDialogueSubsystem>();

for (const FMayDialogueInstanceSnapshot& Snapshot : Save->ActiveDialogueSnapshots)
{
    // Server-only: re-creates an instance and resumes it at the remembered node.
    Subsystem->ResumeDialogueFromSnapshot(Snapshot);
}
```

`ResumeDialogueFromSnapshot` loads the `DialogueAsset` referenced in the snapshot (stored as a soft path — see the table below), creates a new instance, writes back variables and scope stack, and transitions to `CurrentNodeGuid`. If the snapshot was in front of a choice, the choice is re-evaluated and presented.

## Snapshot fields

`FMayDialogueInstanceSnapshot` is the serializable image of an instance.

| Field | Type | Content |
| --- | --- | --- |
| `DialogueAsset` | `FSoftObjectPath` | Which asset was running — as a soft path, so the asset is not held in memory unintentionally. Re-resolved (sync-loaded) on resume. |
| `CurrentNodeGuid` | `FGuid` | Position in the graph: the node at which to resume. |
| `CurrentChoiceNodeGuid` | `FGuid` | The PlayerChoice node that was active (so the resume re-presentation and `ReturnToCurrentChoice` target the right node). |
| `PreviousChoiceNodeGuid` | `FGuid` | The PlayerChoice node active before the current one (so `ReturnToLastChoice` menu chains keep working after resume). |
| `DialogueVariables` | `FInstancedPropertyBag` | The typed dialogue-scope variables (Bool/Int/Float/String/Tag). |
| `ScopeStack` | `TArray<FMayDialogueScopeSnapshotEntry>` | Sub-dialogue nesting for link/return. Each frame stores the parent asset as a soft path plus its return-node GUID. |
| `NodeVisitCounts` | `TMap<FGuid, int32>` | Per-session node visit counts, so Visit Count Gate requirements resolve identically after resume. |
| `InstigatorParticipantTag` | `FGameplayTag` | ParticipantTag of the instigator, used to re-bind to a live actor on resume. |
| `TargetParticipantTag` | `FGameplayTag` | ParticipantTag of the target, used to re-bind to a live actor on resume. |
| `bWasWaitingForChoice` | `bool` | True if the instance was waiting for a player choice at capture time (drives whether resume re-presents the choice or re-enters the node directly). |

The **pending choice list itself is not stored** — on resume the saved PlayerChoice node is re-entered and re-presents a freshly evaluated choice list, so stale choice text can never resurface.

## API reference

| Symbol | Signature | Purpose |
| --- | --- | --- |
| `FMayDialogueInstanceSnapshot` | `USTRUCT` | Serializable image of a running instance (fields above). |
| `UMayDialogueInstance::CreateSnapshot` | `bool CreateSnapshot(FMayDialogueInstanceSnapshot& OutSnapshot) const` | Writes the current state into `OutSnapshot`. `const` — does not mutate the instance. Returns `false` (leaving `OutSnapshot` untouched) when nothing meaningful can be captured. |
| `UMayDialogueSubsystem::ResumeDialogueFromSnapshot` | `UMayDialogueInstance* ResumeDialogueFromSnapshot(const FMayDialogueInstanceSnapshot&)` | **Server only.** Re-creates an instance and resumes it at the remembered node. |
| `UMayDialogueSaveGame::ActiveDialogueSnapshots` | `TArray<FMayDialogueInstanceSnapshot>` | The dialogue snapshots stored in the slot. |
| `UMayDialogueSaveGame::SchemaVersion` | `int32` (now **2**) | On-disk schema version. Bumped to 2 because the save now contains dialogue snapshots. |
| `UMayDialogueSaveHelper::QuickSaveToSlot` | `bool QuickSaveToSlot(WorldContext, SlotName, UserIndex)` | Captures PersistentMemory **and** active dialogue snapshots. |
| `UMayDialogueSaveHelper::QuickLoadFromSlot` | `bool QuickLoadFromSlot(WorldContext, SlotName, UserIndex)` | Restores PersistentMemory **and** resumes saved dialogues. |

## Upgrade & compatibility

The `SchemaVersion` field in `UMayDialogueSaveGame` rises from `1` to `2`, because the save now holds dialogue snapshots.

{% hint style="success" %}
**Old saves still load.** A slot written before the update (schema 1) contains no `ActiveDialogueSnapshots` — migration loads it without a problem, the snapshot array stays empty, and participant memory is restored normally. No save data is lost; only a dialogue that happened to be running at the time of the old save naturally cannot be resumed (it was never captured back then).
{% endhint %}

## Gotchas

| Case | What happens / what to watch for |
| --- | --- |
| **Asset renamed/moved** | `DialogueAsset` is a soft path. If you move the dialogue asset between save and load, resume no longer finds it — keep asset paths stable or use redirectors. |
| **Node deleted from the graph** | If the node referenced by `CurrentNodeGuid` was removed between save and load (graph edited + recompiled), resume cannot dock onto it. The subsystem aborts the affected snapshot cleanly. |
| **Participant missing on load** | If the participants referenced in the snapshot are not loaded back into the world, the dialogue cannot resolve those involved — make sure the NPCs exist before the resume (cf. actor identity in [SaveGame Integration](../persistence/save-integration.md)). |
| **Saved mid-way through an async node** | Expect the node to restart fresh (see above) — not a bug, it's by design. |
| **Resume called on a client** | `ResumeDialogueFromSnapshot` is server-only and a no-op on `NM_Client`. Drive save/load through your server / save-system path. |

> 📸 **Image placeholder:** `resume-snapshot-flow.png` — Flow diagram: active instance → CreateSnapshot → ActiveDialogueSnapshots in the SaveGame → slot. On load: slot → ResumeDialogueFromSnapshot → new instance at CurrentNodeGuid.
> *Setup:* Simple two-lane flow diagram. Top "Save": box "Running instance" → "CreateSnapshot (const)" → "ActiveDialogueSnapshots" → "Slot written (schema 2)". Bottom "Load": "Slot read" → "ResumeDialogueFromSnapshot (server)" → "New instance" → "Transition to CurrentNodeGuid" → "Pending choice re-presented". Arrows clear, steps numbered.
