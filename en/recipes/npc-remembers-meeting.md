---
description: NPC remembers a second meeting – participant variable + SaveGame persistence.
---

# NPC Remembers the 2nd Meeting

## Scenario

An informant reacts with surprise at the first meeting. At the second conversation — even after loading a save — he greets the player by name and remembers the first encounter. To achieve this, you combine a persistent Bool variable with a Branch at the dialogue entry point.

## What You Will Learn

- Create a persistent Bool variable in the participant.
- Set the variable on the first visit (SideEffect on the Exit node).
- Recognize the second visit via Branch.
- How this interacts with the SaveGame system.

## Prerequisites

- [Track Relationship Score](relationship-counter.md) read (variable concept).
- [Persistence → SaveGame Integration](../persistence/save-integration.md) understood.

## Mini-Graph

```text
[Entry]
   │
   ▼
[Branch]
   ├─ BP1: CheckVariable(HasMetBefore = true)
   │       → [SayLine: "Ah, you again! What do you bring me?"] → [PlayerChoice: ...]
   └─ BP2: <Fallback>
           → [SayLine: "Who are you? I don't know you."]
           → [SayLine: "...fine. My name is Corvus."]
           → [PlayerChoice: ...]

All exits:
   └─ SideEffect on Exit node: SetVariable(HasMetBefore = true)
```

> 📸 **Image placeholder:** `npc-remembers-meeting-graph-overview.png` — Graph with Branch for first/return meeting and SideEffect on the Exit.
> *Setup:* Asset `DA_Informant_Corvus` open. Entry → Branch (two outputs). Top path (return meeting): SayLine → PlayerChoice → Exit. Bottom path (first meeting): SayLine × 2 → PlayerChoice → Exit. Every Exit node has a SideEffect pill `SetVariable HasMetBefore = true` visible.

## Step by Step

### 1. Create the Persistent Variable

Asset: `DA_Informant_Corvus`. **Variables panel** → **Add Variable**:

| Property | Value |
|----------|------|
| `Name` | `HasMetBefore` |
| `Type` | `Bool` |
| `Scope` | `Participant` |
| `DefaultValue` | `false` |
| `bPersistent` | `true` |

### 2. Branch for First/Return Meeting

Branch node. BranchPoint[0]: **CheckVariable**:

| Property | Value |
|----------|------|
| `VariableName` | `HasMetBefore` |
| `Scope` | `Participant` |
| `ComparisonOp` | `==` |
| `ComparisonValue` | `true` |

BranchPoint[1]: fallback (no Requirement).

### 3. Return Meeting Path

BranchPoint[0] output → SayLine *"Ah, you again! What do you bring me?"* → PlayerChoice → Exit.

### 4. First Meeting Path

BranchPoint[1] output → SayLine *"Who are you? I don't know you."* → SayLine *"...fine. My name is Corvus."* → PlayerChoice → Exit.

### 5. Set the Variable at Exit

On **every Exit node** in the asset (there may be several), attach a **SideEffect → SetVariable**:
- `Name: HasMetBefore`, `Scope: Participant`, `Value: true`.

This sets the variable regardless of how the conversation ends.

> 📸 **Image placeholder:** `npc-remembers-meeting-exit-sideffect.png` — Exit node with SideEffect pill.
> *Setup:* Exit node selected. In the node body: SideEffect pill `SetVariable: HasMetBefore = true`. Details panel shows the SideEffect properties.

### 6. Enable SaveGame Persistence

On the Participant component of the NPC actor: `bPersistMemory = true`. MayDialogue writes `HasMetBefore` to the SaveGame on dialogue end and loads it when the game starts.

### 7. Test

PIE: start the conversation → first meeting path runs. Save and load the game. Start the conversation again → return meeting path runs.

## Blueprint Triggering

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_Informant_Corvus
   └─ ...
```

> 📸 **Image placeholder:** `npc-remembers-meeting-bp-trigger.png` — Blueprint trigger on the informant.
> *Setup:* Informant actor BP. `Event OnInteract` → `Start Dialogue (MayDialogueLibrary)`. Pins: `Asset = DA_Informant_Corvus`, `Instigator = Get Player Pawn`, `Target = Self`.

## Variations / Going Further

- Multiple meeting counters: `MeetingCount` as Int, increment on every Exit. Unlock a special path after count 3.
- Combine with relationship score: [Track Relationship Score](relationship-counter.md).
- Read the variable from outside: the quest system checks `HasMetBefore` before starting and optionally shows a hint.

## Troubleshooting

**Return meeting path appears on the very first game start.**
`DefaultValue = true` instead of `false`. Check the Variables panel.

**Variable resets after game start.**
`bPersistent = false` or the Participant component has `bPersistMemory = false`. Check both flags.

**SideEffect on Exit doesn't fire.**
The SideEffect is on the wrong node. There are multiple Exit nodes in the asset and not all of them have the SideEffect. Check every Exit node.
