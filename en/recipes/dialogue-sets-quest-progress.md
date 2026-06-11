---
description: Dialogue sets quest tags and fires events – so a conversation drives quest progress forward.
---

# Dialogue Sets Quest Progress

## Scenario

A conversation with a witness should complete the quest step *"Witness interviewed"*. This happens via an AddTag action node and a FireEvent node at the end of the dialogue. The quest system listens for the event and updates its state. No external scripting needed — everything happens inside the dialogue.

## What You Will Learn

- Use the AddTag action node for quest progress tags.
- Use the FireEvent node to notify external systems.
- SideEffect on an Exit node as an alternative to action nodes.
- When to use an action node vs. a SideEffect for quest actions.

## Prerequisites

- [Read Quest Status in Dialogue](quest-status-in-dialogue.md) completed.
- Quest system present that listens for GameplayTag events.

## Mini-Graph

```text
[Entry]
   │
   ▼
[SayLine: "I saw the suspect last night."]
   │
   ▼
[PlayerChoice]
   ├─ "When exactly?" → [SayLine: "Around midnight."]
   └─ "Who was with them?" → [SayLine: "They were alone."]
         │
         └──────────────────────────────┐
                                        ▼
                               [AddTag: Quest.WitnessInterviewed → Player]
                                        │
                                        ▼
                               [FireEvent: Quest.Event.WitnessInterviewed]
                                        │
                                        ▼
                               [Exit: Completed]
```

> 📸 **Image placeholder:** `dialogue-sets-quest-progress-graph-overview.png` — Dialogue graph with AddTag and FireEvent nodes.
> *Setup:* Asset `DA_Witness_Talk` open. SayLine → PlayerChoice with two choices → both paths converge into the same node chain: AddTag node (light green box) → FireEvent node (orange box) → Exit. Layout clearly shows the merging of paths.

## Step by Step

### 1. Define Quest Tags and Event Tags

In `DefaultGameplayTags.ini`:
```ini
+GameplayTagList=(Tag="Quest.WitnessInterviewed")
+GameplayTagList=(Tag="Quest.Event.WitnessInterviewed")
```

### 2. Build the Dialogue Graph

Asset: `DA_Witness_Talk`. Speaker: `Dialogue.Speaker.Witness`. Normal dialogue flow with SayLines and PlayerChoice.

### 3. AddTag Node for Quest Progress

At the end of both choice paths, connections converge into an **AddTag** action node:

| Property | Value |
|----------|------|
| `TagToAdd` | `Quest.WitnessInterviewed` |
| `bAddToInstigator` | `true` |

`bAddToInstigator = true` → the tag is granted to the Instigator's ASC (the player); set it to `false` to grant it to the Target NPC instead. The AddTag node sets a **loose** gameplay tag — it is not, on its own, save-surviving. If the quest flag must persist across saves, apply an Infinite-Duration GameplayEffect with the quest tag in its `GrantedTags` (via the ApplyEffect node) instead.

> 📸 **Image placeholder:** `dialogue-sets-quest-progress-addtag-details.png` — Details panel of the AddTag node.
> *Setup:* AddTag node selected. Details: `TagToAdd = Quest.WitnessInterviewed`, `bAddToInstigator = true`.

### 4. FireEvent Node for External Systems

From the AddTag output → **FireEvent** node:

| Property | Value |
|----------|------|
| `EventTag` | `Quest.Event.WitnessInterviewed` |
| `Payload` | *(optional, leave empty)* |

The FireEvent node broadcasts the event via the MayDialogue event system. External subscribers (quest system, achievement system) receive it via the `OnDialogueEvent` delegate.

### 5. Subscribe the Quest System

```cpp
// In the quest system, on start:
UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(GetWorld());
Sub->OnDialogueEvent.AddUObject(this, &UQuestSystem::HandleDialogueEvent);

// Handler:
void UQuestSystem::HandleDialogueEvent(FGameplayTag EventTag, UMayDialogueInstance* Inst)
{
    if (EventTag == TAG_Quest_Event_WitnessInterviewed)
    {
        AdvanceQuestStep(EQuestStep::WitnessInterviewed);
    }
}
```

### 6. Compile and PIE Test

In PIE: complete the dialogue → check `Quest.WitnessInterviewed` on the Player ASC in the GAS Debugger. Check the quest system log for event reception.

## Action Node vs. SideEffect — When to Use Which

| Situation | Recommendation |
|-----------|------------|
| Quest progress is the main point (prominent in the graph) | AddTag **action node** + FireEvent **action node** |
| Tag set in passing, low visual importance | **SideEffect** on the Exit node |
| Multiple steps in sequence | Action nodes for readability |

For quest progress, action nodes are recommended — at a glance you can see in the graph what the conversation accomplishes.

## Blueprint Alternative: OnDialogueEnded

If you don't want to use FireEvent, you can also react to the dialogue completion in Blueprint:

```text
[On Dialogue Ended (Delegate)]
   ├─ Check: ExitStatus == Completed
   └─► [Quest System → Advance Step: WitnessInterviewed]
```

> 📸 **Image placeholder:** `dialogue-sets-quest-progress-bp-handler.png` — Blueprint handler for OnDialogueEnded.
> *Setup:* Quest Manager Blueprint. `Bind Event to On Dialogue Ended` → `Switch on Exit Status` → `Completed` branch → `Advance Quest Step`. Participant reference via `Get Instigator` from the Instance handle.

## Variations / Going Further

- Use FireEvent with a **Payload**: pass which witness was interviewed (tag as payload).
- **Wait node** that waits for a quest event: dialogue waits for external confirmation → [Wait for GameplayEvent](wait-for-event.md).
- Check quest status before the dialogue → [Read Quest Status in Dialogue](quest-status-in-dialogue.md).

## Troubleshooting

**Tag set, but quest system doesn't react.**
Subscription to `OnDialogueEvent` missing or bound too late. Check the bind order — the subsystem must be subscribed before the dialogue starts.

**FireEvent fires, but EventTag doesn't match.**
`EventTag` in the FireEvent node and in the `HandleDialogueEvent` check are different. Compare tag names exactly.

**AddTag not replicated on Dedicated Server.**
AddTag acts on the local ASC. No problem in singleplayer. For server-authoritative setups: set the tag via RPC on the server.
