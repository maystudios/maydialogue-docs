---
description: Query quest status via GameplayTag in dialogue and deliver different conversation paths.
---

# Read Quest Status in Dialogue

## Scenario

A quest giver reacts differently depending on whether the player hasn't accepted the quest yet, has it active, or has completed it. Quest states are tracked as GameplayTags on the player's ASC. The dialogue Branch reads these tags and chooses the appropriate path.

## What You Will Learn

- Configure multiple HasTag Requirements on a Branch for three states.
- Use BranchPoint prioritization for the correct order.
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

> 📸 **Image placeholder:** `quest-status-in-dialogue-graph-overview.png` — Asset Editor with Branch and three BranchPoints.
> *Setup:* Asset `DA_QuestGiver_Artifact` open. Entry → Branch node (diamond, three output pins). Top pin → short chain SayLine + ApplyEffect → Exit. Middle pin → SayLine → Exit. Bottom pin → SayLine → PlayerChoice → two Exits. All three paths clearly spread out and readable.

## Step by Step

### 1. Define Quest Tags

In `DefaultGameplayTags.ini`:
```ini
+GameplayTagList=(Tag="Quest.FindArtifact.Active")
+GameplayTagList=(Tag="Quest.FindArtifact.Completed")
```

### 2. Branch with Three BranchPoints

Insert a Branch node. Three BranchPoints in this order:

**BranchPoint[0]:** HasTag `Quest.FindArtifact.Completed`, `bCheckOnInstigator: true`.
**BranchPoint[1]:** HasTag `Quest.FindArtifact.Active`, `bCheckOnInstigator: true`.
**BranchPoint[2]:** No Requirement (fallback — quest not yet started).

The order is critical: `Completed` must be checked before `Active`, since a player may carry both tags after finishing the quest.

> 📸 **Image placeholder:** `quest-status-in-dialogue-branch-details.png` — Details panel of the Branch node with three BranchPoints.
> *Setup:* Branch node selected. Details shows: `BranchPoints[0]: Description = "Completed", Requirement: HasTag Quest.FindArtifact.Completed`. `BranchPoints[1]: Description = "Active", Requirement: HasTag Quest.FindArtifact.Active`. `BranchPoints[2]: Description = "Not started", no Requirements`.

### 3. Completion Path (BranchPoint 0)

SayLine *"Excellent! You found the artifact!"* → **ApplyEffect** node (`GE_QuestReward`) → Exit (`Completed`).

To prevent double rewards: on the Exit node, SideEffect `RemoveTag(Quest.FindArtifact.Active)` and `RemoveTag(Quest.FindArtifact.Completed)` — or let the quest system react to `OnDialogueEnded` with status `Completed` and clean up.

### 4. Active Path (BranchPoint 1)

SayLine *"You're still on it. The artifact is south of the tower."* → Exit.

### 5. Not-Started Path (BranchPoint 2)

SayLine *"I need your help."* → PlayerChoice:
- *"Yes, I'll take it."* → SideEffect: `AddTag(Quest.FindArtifact.Active)` on the player → Exit.
- *"Not now."* → Exit.

### 6. Compile and Test

In PIE: without tags → fallback path. Set `Quest.FindArtifact.Active` → middle path. Set `Quest.FindArtifact.Completed` → top path.

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
BranchPoints in the wrong order: `Active` before `Completed`. Swap the order.

**Fallback path runs even though quest is active.**
`bCheckOnInstigator = false` → tag is checked on the NPC. Or a typo in the tag name.

**Reward comes multiple times.**
Completed tag remains on the player after the dialogue. Add RemoveTag at the Exit node, or let the quest system clean up after `OnDialogueEnded`.
