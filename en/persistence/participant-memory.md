---
description: How PersistentMemory on the Participant component works and when to use what.
---

# Participant Memory

`PersistentMemory` on the `UMayDialogueParticipant` component is a typed dictionary that persists across dialogues. It is the right place for things like "has the player already met this NPC?" or "how many times have they been spoken to?".

## Five supported types

| Type | Example variable | Setter | Getter |
| --- | --- | --- | --- |
| `bool` | `HasMet`, `IsAngry` | `SetPersistentBool` | `GetPersistentBool` |
| `int32` | `MeetingCount`, `InsultCount` | `SetPersistentInt` | `GetPersistentInt` |
| `float` | `Reputation`, `LastPrice` | `SetPersistentFloat` | `GetPersistentFloat` |
| `FString` | `LastTopicDiscussed` | `SetPersistentString` | `GetPersistentString` |
| `FGameplayTag` | `LastChoseTag` | `SetPersistentTag` | `GetPersistentTag` |

All getters accept a `DefaultValue` parameter: if the variable does not yet exist, the default is returned (no crash, no log warning).

{% hint style="info" %}
**Variable does not exist?** On write it is created automatically. On read the default is returned. You never need to check whether a variable exists beforehand.
{% endhint %}

## Reading and writing in dialogue

The simplest approach: use `SetVariable` nodes and `GetVariable` nodes directly in the dialogue with Scope set to `Participant`. Everything then happens declaratively in the dialogue asset, without any Blueprint glue.

> 📸 **Image placeholder:** `memory-set-in-dialogue.png` — SetVariable SideEffect on a SayLine, Scope = Participant.
> *Setup:* SayLine "Nice to meet you!" selected in the MayDialogue Editor. SideEffects array expanded: `Set Variable` sub-node visible, `Variable Name = HasMet`, `Scope = Participant`, `Type = Bool`, `Value = true`. Details panel on the right shows the same fields.

## Reading and writing from Blueprint code

If your quest system, achievement system, or another actor needs to set memory state:

```text
[Get Component by Class] → UMayDialogueParticipant
  │
  ├─ [Set Persistent Bool]   VarName="HasMet"        Value=true
  ├─ [Set Persistent Int]    VarName="MeetingCount"  Value=GetPersistentInt("MeetingCount")+1
  └─ [Get Persistent Bool]   VarName="HasMet"        Default=false  →  Return value
```

> 📸 **Image placeholder:** `memory-blueprint-getset.png` — Blueprint graph: get the Participant component, call SetPersistentBool and GetPersistentInt.
> *Setup:* Blueprint graph in the NPC Blueprint. `Get Component by Class` → `Set Persistent Bool` (VarName="HasMet", Value=true). Second chain: `Get Persistent Int` (VarName="MeetingCount", Default=0) → `Integer + 1` → `Set Persistent Int` (VarName="MeetingCount"). All pins connected.

## When to use what

| Situation | Recommendation |
| --- | --- |
| Remember first contact with NPC | `SetPersistentBool("HasMet", true)` on first dialogue end |
| Count number of conversations | `SetPersistentInt("MeetingCount", GetPersistentInt+1)` |
| Last conversation topic for a follow-up dialogue | `SetPersistentString("LastTopic", …)` |
| Quest stage of an NPC conversation path | `SetPersistentInt("QuestPhase", …)` |
| Complex structs, inventory snapshots | Own `UPROPERTY(SaveGame)` in a Participant Blueprint subclass |

{% hint style="warning" %}
**Type mismatch:** If you call `GetPersistentBool` on a variable that was stored as an Int, you receive the default value — no conversion takes place. Keep the type and usage site consistent.
{% endhint %}

## Event: OnVariableChanged

The component broadcasts `OnVariableChanged` every time a persistent variable is set:

```text
[Participant Component]
  OnVariableChanged (VarName, Scope, Type, NewValueAsString)
    │
    └─ Bind in your system Blueprint
         → Check VarName == "IsAngry" and NewValue == "true"
         → Trigger Combat / AI state change
```

> 📸 **Image placeholder:** `memory-event-binding.png` — Blueprint: bind OnVariableChanged delegate and react to "IsAngry".
> *Setup:* Blueprint graph in the NPC Blueprint, `BeginPlay` event. `Get MayDialogue Participant Component` → `Bind Event to On Variable Changed` → custom event `OnVarChanged` with parameters `VarName (Name), Scope, Type, NewValue (String)`. In the event body: `Branch`: VarName == "IsAngry" AND NewValue == "true" → `Set AI State: Combat`.

## Lifecycle

| Point in time | What happens to PersistentMemory |
| --- | --- |
| Actor spawn | PropertyBag is empty (or loaded from SaveGame) |
| During dialogue | Getters/setters active via nodes or code |
| Actor destroy | Memory is lost if not saved beforehand |
| After QuickSave | Memory written to slot |
| After QuickLoad | Memory restored from slot |

## More complex data

The PropertyBag only supports the five basic types. For more complex structures (e.g. a dictionary of item IDs), create a **Blueprint subclass of `UMayDialogueParticipant`** and extend it with your own variables:

- **In Blueprint:** Create your own variables in the variable details view and enable the `Save Game` checkbox for each. These variables are automatically included in UE's standard SaveGame archive.
- **In C++:** Add your own `UPROPERTY(SaveGame)` fields to the subclass — the result is the same.

Then set this subclass as the Participant class for your actor.
