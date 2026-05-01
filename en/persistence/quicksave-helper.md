---
description: Save and load Participant memory without your own SaveGame system.
---

# QuickSave Helper

For projects without their own SaveGame system, MayDialogue provides `UMayDialogueSaveHelper` — a Blueprint Function Library with four functions. With it you can save and load the PersistentMemory of all participants with a single function call.

## Core functions

| Function | What it does |
| --- | --- |
| `QuickSaveToSlot(SlotName, UserIndex)` | Collects PersistentMemory from all participants in the world and writes it to the slot. |
| `QuickLoadFromSlot(SlotName, UserIndex)` | Reads the slot and distributes memory to all participants with a matching ParticipantTag. |
| `DeleteSlot(SlotName, UserIndex)` | Deletes a slot. |
| `DoesSlotExist(SlotName, UserIndex)` | Checks whether a slot exists (call before loading). |

All are available in Blueprint (category `MayDialogue|Persistence`).

## GlobalMemory helpers

For project-wide flags (e.g. "has the player seen the intro?") simple getters and setters are available on the `GlobalMemory` field — without direct access to the `FInstancedPropertyBag` container:

| Function | Type | Description |
| --- | --- | --- |
| `GetGlobalBool(SlotName, Key, Default)` | `bool` | Read a bool flag |
| `SetGlobalBool(SlotName, Key, Value)` | — | Write a bool flag |
| `GetGlobalInt(SlotName, Key, Default)` | `int32` | Read an integer value |
| `SetGlobalInt(SlotName, Key, Value)` | — | Write an integer value |
| `GetGlobalFloat(SlotName, Key, Default)` | `float` | Read a float value |
| `SetGlobalFloat(SlotName, Key, Value)` | — | Write a float value |
| `GetGlobalString(SlotName, Key, Default)` | `FString` | Read a string value |
| `SetGlobalString(SlotName, Key, Value)` | — | Write a string value |

All GlobalMemory helpers are Blueprint-callable (category `MayDialogue|Persistence|Global`).

```text
[Set Global Float]   (e.g. at the end of a companion dialogue)
  ├─ Slot Name: "AutoSave"
  ├─ Key:       "CompanionAffection"
  └─ Value:     0.75
```

> **Note:** Global and participant-specific memory data cannot be inspected directly in Blueprint — use the `Get Persistent ...` / `Set Persistent ...` functions on the Participant and the `Get Global ...` / `Set Global ...` nodes of the SaveHelper library instead.

> 📸 **Image placeholder:** `quicksave-blueprint-save.png` — Blueprint graph: Level Exit event → QuickSaveToSlot.
> *Setup:* Blueprint graph in the Level Blueprint. `Event Level Exit` → `Does Slot Exist` (SlotName="AutoSave") → `Quick Save To Slot` (WorldContextObject=Self, SlotName="AutoSave", UserIndex=0) → `Print String "Saved"`. All nodes connected, return value (bool) ignored via dummy branch.

## Saving — example

```text
[Level Exit Event]
  │
  └─ [Quick Save To Slot]
       SlotName:  "AutoSave"
       UserIndex: 0
       → returns bool (success)
```

## Loading — example

```text
[Begin Play]
  │
  ├─ [Does Slot Exist? "AutoSave"]
  │     ├─ true  → [Quick Load From Slot "AutoSave"]
  │     └─ false → (no action — first launch)
```

> 📸 **Image placeholder:** `quicksave-blueprint-load.png` — Blueprint graph: BeginPlay → DoesSlotExist → QuickLoadFromSlot with Branch.
> *Setup:* Blueprint graph in the GameMode or Level Blueprint. `Event Begin Play` → `Does Slot Exist` (SlotName="AutoSave") → `Branch`. True path: `Quick Load From Slot` (SlotName="AutoSave"). False path: no node (empty / "first run" comment). Clear path, easy to read.

## How matching works

`QuickLoadFromSlot` distributes memory by `ParticipantTag`. Participants in the saved slot and participants in the currently loaded world are matched by tag name. Only participants whose `ParticipantTag` is present in the slot receive their data back.

{% hint style="warning" %}
**ParticipantTag must be stable.** If you change the `ParticipantTag` of an NPC between save and load, the load path finds no match. Keep tags consistent.
{% endhint %}

## Limitations

| Limitation | Impact |
| --- | --- |
| MayDialogue data only | Inventory, player position, etc. are not saved |
| No versioning | After a PropertyBag structure update, old slots may be incompatible |
| No delta save | Every save rewrites all participant data |
| No multi-slot management | You manage multiple slots yourself via SlotName |

## When to use the QuickSave Helper vs. your own system

| Scenario | Recommendation |
| --- | --- |
| Game jam or prototype | QuickSave Helper — ready to use immediately |
| Small indie project without complex state | QuickSave Helper as a starting point, migrate if needed |
| Project with inventory, world state, quests | Own SaveGame system, embed `PersistentMemory` via `ArIsSaveGame` (→ [SaveGame Integration](save-integration.md)) |

## Reading saved participant tags

With `GetSavedParticipantTags(SlotName, UserIndex)` (BlueprintPure) you can retrieve the list of all saved participant tags from a slot — useful for slot preview UIs (e.g. "how many NPCs have been spoken to?"):

```text
[Get Saved Participant Tags]
  ├─ Slot Name: "AutoSave"
  ├─ User Index: 0
  └─ Return: TArray<FGameplayTag>
```

> 📸 **Image placeholder:** `quicksave-slot-naming.png` — Blueprint graph: dynamic SlotName from "Save_" + CurrentLevel.
> *Setup:* Blueprint graph. `Get Current Level Name` → `Append "Save_"` → `Quick Save To Slot` with the composed SlotName. Shows how you can use a separate slot per level.
