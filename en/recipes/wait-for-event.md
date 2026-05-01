---
description: Dialogue pauses and waits for an external GameplayEvent – perfect for synchronized cutscene moments.
---

# Wait for External GameplayEvent

## Scenario

An NPC says *"Wait, let me show you something."* — then a door opens (external system, Blueprint) and only after that does the NPC continue talking. The **Wait** node pauses the dialogue flow until a `GameplayTag` event arrives. The external system fires this event when the door is open.

## What You Will Learn

- Configure a Wait node for `EventTag`-based waiting.
- Fire a GameplayEvent from outside into the dialogue (Blueprint + C++).
- Combine with a Duration fallback (maximum wait time).
- Timeout behavior when the event never arrives.

## Prerequisites

- [Simple NPC Conversation](simple-npc-talk.md) completed.
- External system (door Blueprint) can fire GameplayEvents.

## Mini-Graph

```text
[Entry]
   │
   ▼
[SayLine: NPC – "Wait, let me show you something. That door over there..."]
   │
   ▼
[Wait: EventTag=Interaction.DoorOpened  MaxDuration=10s  TimeoutBehavior=Continue]
   │
   ▼
[SayLine: NPC – "You see? The secret lies beyond."]
   │
   ▼
[Exit: Completed]
```

> 📸 **Image placeholder:** `wait-for-event-graph-overview.png` — Dialogue graph with Wait node between two SayLines.
> *Setup:* Asset `DA_NPC_ShowSecret` open. SayLine → Wait node (clock icon in title, gray) → SayLine → Exit. Wait node in the Details panel: `EventTag = Interaction.DoorOpened`, `MaxDuration = 10.0`, `TimeoutBehavior = Continue`.

## Step by Step

### 1. Define the Event Tag

In `DefaultGameplayTags.ini`:
```ini
+GameplayTagList=(Tag="Interaction.DoorOpened")
```

### 2. Insert the Wait Node

Asset: `DA_NPC_ShowSecret`. After the first SayLine → **Create Node → Wait**:

| Property | Value |
|----------|------|
| `bWaitForEvent` | `true` |
| `EventTag` | `Interaction.DoorOpened` |
| `bWaitForDuration` | `false` |
| `MaxDuration` | `10.0` |
| `TimeoutBehavior` | `Continue` |

`TimeoutBehavior = Continue` → after MaxDuration, the dialogue continues even if the event never came.

> 📸 **Image placeholder:** `wait-for-event-node-details.png` — Details panel of the Wait node.
> *Setup:* Wait node selected. Details: `bWaitForEvent = true`, `EventTag = Interaction.DoorOpened`, `MaxDuration = 10.0`, `TimeoutBehavior = Continue`. `bWaitForDuration = false`.

### 3. Wire Up the Dialogue Flow

Wait output → SayLine *"You see? The secret lies beyond."* → Exit.

### 4. External System Fires the Event

**Blueprint – Door Actor:**

```text
[Event On Door Fully Opened]
   │
   ▼
[MayDialogueSubsystem → Fire Dialogue Event]
   ├─ EventTag: Interaction.DoorOpened
   └─ Context:  (optional, door actor)
```

> 📸 **Image placeholder:** `wait-for-event-bp-door.png` — Blueprint of the door actor with FireDialogueEvent call.
> *Setup:* Door actor Blueprint. `On Door Fully Opened` (custom event or timeline end) → `Get May Dialogue Subsystem` → `Fire Dialogue Event`. Pins: `EventTag = Interaction.DoorOpened`.

**C++:**

```cpp
// In the door Blueprint or Actor, when the door is fully open:
if (UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(GetWorld()))
{
    Sub->FireDialogueEvent(TAG_Interaction_DoorOpened, nullptr);
}
```

### 5. Compile and Test in PIE

In PIE: start the dialogue → NPC says the first line → dialogue pauses at the Wait. Open the door → event fires → dialogue continues. Let MaxDuration expire without the event → dialogue continues automatically after 10s.

## Wait Modes at a Glance

| Mode | Configuration | Behavior |
|-------|---------------|-----------|
| Event only | `bWaitForEvent = true`, `bWaitForDuration = false` | Waits indefinitely until event arrives |
| Duration only | `bWaitForEvent = false`, `bWaitForDuration = true` | Waits N seconds, then continues |
| Event OR Duration | `bWaitForEvent = true`, `bWaitForDuration = true` | Whichever comes first |
| Event AND Duration | Not directly, but: event valid only after timer | Via FireEvent with Delay |

## Timeout Behavior Options

| Option | Behavior at MaxDuration |
|--------|--------------------------|
| `Continue` | Dialogue continues as if the event had arrived |
| `Abort` | Dialogue aborts (ExitStatus = Failed) |
| `LoopBack` | Restart the Wait node (caution: can cause an infinite loop) |

## Variations / Going Further

- **Player action as trigger**: instead of a door, a player click on an object fires the event.
- **Dialogue fires its own event**: FireEvent node + Wait node in combination for a ping-pong between the dialogue and the game world.
- **Cinematic sync**: LevelSequence ends → event fires → dialogue flow continues → perfect transition.

## Troubleshooting

**Dialogue gets stuck, event never arrives.**
The external system is not calling `FireDialogueEvent`, or it uses a different EventTag. Compare tag names exactly. Set MaxDuration as a safety net.

**Event arrives too early (before the Wait node).**
The Wait node is not yet active. Events that arrive before the Wait node is entered are ignored. Make sure the door opening happens after the Wait node is entered.

**TimeoutBehavior = Abort, but no ExitStatus visible.**
Check the `OnDialogueEnded` delegate — ExitStatus is `Failed` on abort. The quest system must handle this case.
