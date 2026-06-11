---
description: How to wire MayDialogue into your own SaveGame system.
---

# SaveGame Integration

If your project already has a SaveGame system, you slot MayDialogue data in seamlessly. The Participant component is already prepared for it.

## Blueprint Quick Start

For projects without their own SaveGame system, `UMayDialogueSaveHelper` is ready to go — a Blueprint Function Library with simple save and load nodes.

**Saving** (e.g. when leaving a level):

```text
[Event Level Exit]
    │
    ▼
[Quick Save To Slot]  (Category: MayDialogue|Persistence)
    ├─ WorldContextObject: Self
    ├─ Slot Name:          "AutoSave"
    └─ User Index:         0
         │ Return Value (bool) — true on success
```

**Loading** (e.g. when the next level starts):

```text
[Event Begin Play]
    │
    ▼
[Does Slot Exist]  (Category: MayDialogue|Persistence)
    ├─ Slot Name: "AutoSave"
    │ True
    ▼
[Quick Load From Slot]  (Category: MayDialogue|Persistence)
    ├─ WorldContextObject: Self
    ├─ Slot Name:          "AutoSave"
    └─ User Index:         0
```

`Quick Save To Slot` automatically gathers the `PersistentMemory` of every `UMayDialogueParticipant` component in the world and writes it into a UE SaveGame slot. `Quick Load From Slot` restores it on load — participants are identified by their `ParticipantTag`.

In addition, you can store project-wide flags (with no participant association) via `Set Global Bool` / `Get Global Bool` and the corresponding Int, Float and String variants in the category `MayDialogue|Persistence|Global`.

---

## Advanced Integration (C++)

If your project has its own SaveGame system, you slot MayDialogue data straight into your save pipeline. The Participant component is already prepared.

## The UPROPERTY(SaveGame) Flag

On `UMayDialogueParticipant`:

```cpp
UPROPERTY(SaveGame)
FInstancedPropertyBag PersistentMemory;
```

The `SaveGame` specifier tells UE's serializer: "include this property when serializing a SaveGame archive." You don't have to do anything else, as long as your archive sets `ArIsSaveGame = true`.

## Step 1 — On Save

When your project saves, you iterate all participants and serialize the actor:

```cpp
// In your SaveSystem or SaveGame handler:
TArray<AActor*> Participants;
UGameplayStatics::GetAllActorsWithComponent(World, UMayDialogueParticipant::StaticClass(), Participants);

for (AActor* Actor : Participants)
{
    TArray<uint8> Bytes;
    FMemoryWriter Writer(Bytes);
    FObjectAndNameAsStringProxyArchive Ar(Writer, /*bLoadIfFindFails=*/false);
    Ar.ArIsSaveGame = true;

    Actor->Serialize(Ar);  // PersistentMemory is written too, because of the SaveGame flag
    // Pack Bytes into your SaveData struct, key = actor name or GUID
}
```

> 📸 **Image placeholder:** `saveint-blueprint-save.png` — Blueprint graph: iterate all actors with UMayDialogueParticipant and serialize them.
> *Setup:* BP graph in the SaveSystem Blueprint. `Get All Actors With Component` (ComponentClass = MayDialogueParticipant) → ForEach loop → `Save Actor To Bytes` function (from your own utility library). The loop output feeds into a `SaveData` array. No MayDialogue-specific node needed — standard UE SaveGame path.

## Step 2 — On Load

The inverse direction: apply the saved bytes back onto the actors.

```cpp
for (auto& [ActorName, Bytes] : SavedData)
{
    AActor* Actor = FindActorByName(World, ActorName);  // your lookup logic
    if (!Actor) continue;

    FMemoryReader Reader(Bytes);
    FObjectAndNameAsStringProxyArchive Ar(Reader, false);
    Ar.ArIsSaveGame = true;

    Actor->Serialize(Ar);  // PersistentMemory is restored
}
```

`FInstancedPropertyBag` knows its own schema and deserializes itself correctly, as long as the schema is unchanged between save and load.

## Manual Approach (Alternative)

You don't have to use the standard archive path. You can also pull the PropertyBag straight off the participant and stick it into your own SaveGame struct:

```cpp
// On save:
UMayDialogueParticipant* Part = Actor->FindComponentByClass<UMayDialogueParticipant>();
FInstancedPropertyBag Memory = Part->PersistentMemory;
MySaveData.DialogueMemories.Add(Part->ParticipantTag.GetTagName(), Memory);

// On load:
FInstancedPropertyBag LoadedMemory = MySaveData.DialogueMemories.FindRef(Part->ParticipantTag.GetTagName());
Part->PersistentMemory = LoadedMemory;
```

This gives you full control over when the sync happens.

> 📸 **Image placeholder:** `saveint-manual-bp.png` — Blueprint graph: manual Get/Set PersistentMemory.
> *Setup:* BP graph showing `Get Component by Class` (MayDialogueParticipant) → `Get Persistent Memory` → `Add to SaveData Map`. Second part: `Load from Map` → `Set Persistent Memory`. Both parts in the same graph, labelled with comment boxes "On Save" and "On Load".

## What Exactly Gets Saved

For each participant, the `PersistentMemory` PropertyBag is saved — a typed dictionary of Bool, Int, Float, String and GameplayTag values. Every variable you wrote via `SetPersistentBool()`, `SetPersistentInt()` etc. lives in there.

**Not** saved:

* Running dialogue instances (they are restarted if needed).
* Dialogue-scope variables.
* UI state.

## Actor Identity on Load

PersistentMemory only survives if the Participant component lands on the right actor on load. Typical pattern:

1. The level contains NPC "Guard_01" (placed in the scene).
2. On save: actor name or GUID + PersistentMemory are saved.
3. On load: the level loads, the NPC reappears, PersistentMemory is written back by your system via the name/GUID.

{% hint style="warning" %}
**Dynamically spawned NPCs:** if an actor is not placed in the level but spawned dynamically, make sure the spawn path reuses the same `ParticipantTag` and a stable identity (e.g. a saved GUID).
{% endhint %}

## GlobalMemory

`UMayDialogueSaveGame` additionally has a `GlobalMemory` field (`FInstancedPropertyBag`). Use it for project-wide dialogue flags that don't belong to any specific participant, e.g. *"Has the player seen the tutorial conversation?"*. You access `UMayDialogueSaveGame::GlobalMemory` from code and save it alongside the participant data.

> 📸 **Image placeholder:** `saveint-flow-diagram.png` — Timeline: user presses Save → system iterates participants → archive serializes → slot written. On load: slot read → actor found → archive deserialized → PersistentMemory restored.
> *Setup:* Simple flow diagram with two paths: "Save" (top) and "Load" (bottom). Save: box "User presses Save" → "Iterate Participants" → "Serialize ArIsSaveGame=true" → "Write Slot". Load: "Load Slot" → "Find Actor by Name" → "Deserialize" → "PersistentMemory restored". Arrows clear, steps numbered.
