---
description: Query quest status via GameplayTag in dialogue and deliver different conversation paths.
---

# Read Quest Status in Dialogue

## Scenario

A quest giver reacts differently depending on whether the player hasn't accepted the quest yet, has it active, or has completed it. Quest states are tracked as GameplayTags on the player's ASC. The dialogue Branch reads these tags and chooses the appropriate path.

## What You Will Learn

- Chain two Branch nodes to route three quest states.
- Order the checks correctly (Completed before Active).
- Read quest tags on the player's ASC (tag convention).

## Prerequisites

- [Branching with Conditions](branching-conditions.md) completed.
- Quest system sets tags: `Quest.FindArtifact.Completed`, `Quest.FindArtifact.Active`.

## Mini-Graph

```text
[Entry]
   │
   ▼
[Branch]
   ├─ BP1: HasTag(Quest.FindArtifact.Completed)
   │       → [SayLine: "Excellent! You found the artifact! Here is your reward."] → [ApplyEffect: GE_QuestReward] → [Exit: Completed]
   ├─ BP2: HasTag(Quest.FindArtifact.Active)
   │       → [SayLine: "You're still on it. The artifact is south of the tower."] → [Exit]
   └─ BP3: <Fallback>
           → [SayLine: "I need your help. Do you want a job?"] → [PlayerChoice: Accept/Decline]
```

> 📸 **Image placeholder:** `quest-status-in-dialogue-graph-overview.png` — Asset Editor with two chained Branch nodes.
> *Setup:* Asset `DA_QuestGiver_Artifact` open. Entry → Branch A (Condition: HasTag Completed). Branch A True → short chain SayLine + ApplyEffect → Exit. Branch A False → Branch B (Condition: HasTag Active). Branch B True → SayLine → Exit. Branch B False → SayLine → PlayerChoice → two Exits. All three paths clearly spread out and readable.

## Step by Step

### 1. Define Quest Tags

In `DefaultGameplayTags.ini`:
```ini
+GameplayTagList=(Tag="Quest.FindArtifact.Active")
+GameplayTagList=(Tag="Quest.FindArtifact.Completed")
```

### 2. Two Chained Branch Nodes (three outcomes)

A Branch evaluates a single `Condition` and routes to **True** / **False** (plus an optional Default). For a three-way *completed / active / not-started* split, **chain two Branch nodes**:

**Branch A** — `Condition` = HasTag `Quest.FindArtifact.Completed`, `bCheckOnInstigator: true`.
- **True** → completion path (step 3).
- **False** → **Branch B**.

**Branch B** — `Condition` = HasTag `Quest.FindArtifact.Active`, `bCheckOnInstigator: true`.
- **True** → active path (step 4).
- **False** → not-started path (step 5).

The order is critical: check `Completed` **first** (Branch A), since a player may carry both tags after finishing the quest — otherwise the active path would swallow a completed quest.

> 📸 **Image placeholder:** `quest-status-in-dialogue-branch-details.png` — Two chained Branch nodes in the graph.
> *Setup:* Entry → Branch A (Condition: HasTag Quest.FindArtifact.Completed). Branch A True → completion path; Branch A False → Branch B (Condition: HasTag Quest.FindArtifact.Active). Branch B True → active path; Branch B False → not-started path. All connections visible.

### 3. Completion Path (Branch A → True)

SayLine *"Excellent! You found the artifact!"* → **ApplyEffect** node (`GE_QuestReward`) → Exit (`Completed`).

To prevent double rewards: on the Exit node, SideEffect `RemoveTag(Quest.FindArtifact.Active)` and `RemoveTag(Quest.FindArtifact.Completed)` — or let the quest system react to `OnDialogueEnded` with status `Completed` and clean up.

### 4. Active Path (Branch B → True)

SayLine *"You're still on it. The artifact is south of the tower."* → Exit.

### 5. Not-Started Path (Branch B → False)

SayLine *"I need your help."* → PlayerChoice:
- *"Yes, I'll take it."* → SideEffect: `AddTag(Quest.FindArtifact.Active)` on the player → Exit.
- *"Not now."* → Exit.

### 6. Compile and Test

In PIE: without tags → not-started path. Set `Quest.FindArtifact.Active` → active path. Set `Quest.FindArtifact.Completed` → completion path.

## Blueprint Triggering

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_QuestGiver_Artifact
   └─ ...
```

> 📸 **Image placeholder:** `quest-status-in-dialogue-bp-trigger.png` — Blueprint with Start Dialogue and tags in the ASC Debugger.
> *Setup:* Quest giver BP with Start Dialogue. Next to it: GAS Debugger panel with `Quest.FindArtifact.Active` marked on the Player ASC.

## Variations / Going Further

- Track quest status as a participant variable instead of a tag → [Track Relationship Score](relationship-counter.md) shows the pattern.
- Dialogue sets quest tags itself → [Dialogue Sets Quest Progress](dialogue-sets-quest-progress.md).
- Give rewards inside the dialogue → [Apply GameplayEffect from Dialogue](apply-gameplay-effect.md).

## Troubleshooting

**Completion path never reached, even though Completed tag is set.**
The Branches are chained in the wrong order: `Active` (Branch A) before `Completed`. Make the **Completed** check the first Branch, since a finished quest may carry both tags.

**Not-started path runs even though quest is active.**
`bCheckOnInstigator = false` → tag is checked on the NPC. Or a typo in the tag name.

**Reward comes multiple times.**
Completed tag remains on the player after the dialogue. Add RemoveTag at the Exit node, or let the quest system clean up after `OnDialogueEnded`.
