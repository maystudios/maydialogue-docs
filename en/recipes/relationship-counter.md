---
description: Track a relationship score persistently on the participant and react to it in dialogue.
---

# Track Relationship Score

## Scenario

You want an important NPC to remember how often and how the player has spoken with them. A `Relationship` int value rises with friendly answers and falls with rude ones. Once the value reaches 10, the NPC unlocks a quest. This score survives save games because it lives in the participant's memory.

## What You Will Learn

- Create a participant variable (Scope: Participant, Int).
- Configure a SetVariable SideEffect for increment/decrement.
- Use a CheckVariable Requirement to react to the score.
- Persistently save participant memory.

## Prerequisites

- [GAS-Driven Dialogue](gas-driven-dialogue.md) completed.
- [Persistence → Participant Memory](../persistence/participant-memory.md) read.

## Mini-Graph

```text
[Entry]
   │
   ▼
[Branch]
   ├─ BP1: CheckVariable(Relationship >= 10) → [SayLine: "I trust you now. Here is the quest."] → [Exit: QuestGranted]
   └─ BP2: <Fallback>                         → [SayLine: "How can I help you?"] → [PlayerChoice]

[PlayerChoice]
   ├─ "Reply kindly."     → SideEffect: Relationship += 2   → [Exit]
   ├─ "Reply neutrally."  → SideEffect: Relationship += 1   → [Exit]
   └─ "Reply rudely."     → SideEffect: Relationship -= 1   → [Exit]
```

> 📸 **Image placeholder:** `relationship-counter-graph-overview.png` — Complete relationship dialogue in the Asset Editor.
> *Setup:* Asset `DA_Advisor_Talk` open. Entry → Branch (two outputs). Top path: SayLine "Quest unlocked" → Exit. Bottom path: SayLine → PlayerChoice with three choices, each with a SideEffect pill visible on the Choice sub-node (Relationship += 2 / += 1 / -= 1).

## Step by Step

### 1. Create the Participant Variable

Asset: `DA_Advisor_Talk`. Open the **Variables panel** → **Add Variable**:

| Property | Value |
|----------|------|
| `Name` | `Relationship` |
| `Type` | `Int` |
| `Scope` | `Participant` |
| `DefaultValue` | `0` |
| `bPersistent` | `true` |

`Scope: Participant` = the variable lives on the NPC's Participant component, not just for the duration of the dialogue.

> 📸 **Image placeholder:** `relationship-counter-variables-panel.png` — Variables panel with the Relationship variable.
> *Setup:* Variables panel tab open. Single entry: `Relationship | Int | Participant | Default: 0 | Persistent: true`.

### 2. Branch with CheckVariable

Insert a Branch node and set its `Condition` to a **CheckVariable** requirement:

| Property | Value |
|----------|------|
| `VariableName` | `Relationship` |
| `VariableType` | `Int` |
| `Scope` | `Participant` |
| `ComparisonOp` | `>=` |
| `IntValue` | `10` |

The Branch's **True** output runs when the condition passes (relationship ≥ 10); the **False** output runs otherwise. Enable `bHasFallback` only if you also need a third degenerate path.

### 3. PlayerChoice with Relationship SideEffects

Create a PlayerChoice node. Three choices:

**Choice 1** (Friendly): On the Choice sub-node **SideEffect → SetVariable**:
- `Name: Relationship`, `Op: Increment`, `Value: 2`.

**Choice 2** (Neutral): SideEffect with `Op: Increment`, `Value: 1`.

**Choice 3** (Rude): SideEffect with `Op: Decrement`, `Value: 1`.

All three choices → Exit.

### 4. Activate Persistence

The NPC actor's Participant component must have `bPersistMemory = true`. MayDialogue then writes the variable to the SaveGame on dialogue end. See [Participant Memory](../persistence/participant-memory.md) for details.

### 5. Compile and Test

In PIE: three conversations with a friendly answer → Relationship = 6. Two more → Relationship = 10 → next time you open the dialogue, the quest path triggers.

## Reading the Relationship Value from Outside

```text
[Get Participant Variable (Int)]
   ├─ Participant:    NPC Actor Component
   ├─ VariableName:   "Relationship"
   └─► Return Value → Use in Quest System
```

```cpp
int32 Score = NPC->GetParticipantComponent()->GetVariableInt(TEXT("Relationship"));
```

> 📸 **Image placeholder:** `relationship-counter-bp-read.png` — Blueprint graph for reading the Relationship value.
> *Setup:* Blueprint of a quest system. `Get Participant Component (NPC)` → `Get Variable Int ("Relationship")` → `Branch (>= 10)` → Quest-grant logic.

## Variations / Going Further

- Multiple relationship variables for different factions: `Rel_Guards`, `Rel_Merchants`, `Rel_Rebels`.
- Display the relationship score in the HUD: query it from outside via the [Read/Write API](../runtime/read-write-api.md).
- Negative threshold: at Relationship <= -5 → NPC refuses further conversations (`FailedAndHidden` on all choices).

## Troubleshooting

**Variable resets on every game start.**
`bPersistent = false` or the Participant component has `bPersistMemory = false`. Check both.

**Branch never takes the quest path, even though score >= 10.**
`Scope` in the CheckVariable Requirement is `Dialogue` instead of `Participant`. Dialogue scope only exists for the runtime of the dialogue and is 0 after every start.

**SideEffect Increment doesn't appear to happen.**
SideEffects fire when the node is *entered*. For choices, the SideEffect fires when the choice is *selected* and its output node is entered — not when the choice list is displayed.
