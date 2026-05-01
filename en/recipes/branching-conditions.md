---
description: Combine Branch node and GAS Requirement – dialogue reacts to the player's state.
---

# Branching with Conditions

## Scenario

A guard at the city gate reacts differently depending on whether the player has the tag `Story.Pass.CityGate`. With a pass, the player gets through; without one, they are turned away and receive a PlayerChoice to resolve the situation. This is the standard pattern for world reactivity in MayDialogue.

## What You Will Learn

- Create a Branch node with multiple BranchPoints.
- Configure a HasTag Requirement on a BranchPoint.
- Use the fallback BranchPoint as the default path.
- The difference between Branch (entire path) and Choice Requirement (individual option).

## Prerequisites

- [Simple NPC Conversation](simple-npc-talk.md) completed.
- GAS active in the project, PlayerController has an ASC.

## Mini-Graph

```text
[Entry]
   │
   ▼
[Branch]
   ├─ BP1: HasTag(Story.Pass.CityGate) → [SayLine: "Welcome back."] → [Exit: Completed]
   └─ BP2: <Fallback>                  → [SayLine: "Halt! Who goes there?"] → [PlayerChoice]
                                                                               ├─ "I have no pass." → [Exit: Failed]
                                                                               └─ "I'm looking for the gatekeeper." → [SayLine: "Chamber on the left."] → [Exit: Completed]
```

> 📸 **Image placeholder:** `branching-conditions-graph-overview.png` — Overview graph of the guard dialogue.
> *Setup:* Asset `DA_Guard_Gate` open. Visible: `Entry` → `Branch` node (diamond shape, two output pins). Top pin leads to `SayLine "Welcome back"` → `Exit`. Bottom pin leads to `SayLine "Halt!"` → `PlayerChoice` with two Choice sub-nodes → two outputs to one Exit each. Horizontal layout, no comment block.

## Step by Step

### 1. Create the Asset and Speaker

Asset: `DA_Guard_Gate`. Speaker in the Speakers panel: `DisplayName = Guard`, `SpeakerTag = Dialogue.Speaker.Guard`.

### 2. Insert the Branch Node

From the Entry output → **Create Node → Branch**. The Branch node appears in the graph as a diamond shape with one input and multiple output pins.

### 3. Configure the First BranchPoint

Select the Branch node. In the Details panel under `BranchPoints`:
- **Add** → `Description: "With Pass"`.
- Under Requirements: **Add → HasTag**.

| Property | Value |
|----------|------|
| `RequiredTag` | `Story.Pass.CityGate` |
| `bCheckOnInstigator` | `true` (checks the player, not the NPC) |

> 📸 **Image placeholder:** `branching-conditions-branch-details.png` — Details panel of the Branch node with HasTag Requirement.
> *Setup:* Branch node selected. Details panel shows: `BranchPoints[0].Description = "With Pass"`, below it the expanded HasTag Requirement with `RequiredTag = Story.Pass.CityGate`, `bCheckOnInstigator = true`. BranchPoints[1] without Requirements (fallback).

### 4. Second BranchPoint as Fallback

**Add** → `Description: "Fallback"`. Leave Requirements empty. This point activates whenever no earlier one matches.

### 5. Wire Up the Success Path

BranchPoint[0] output pin → SayLine *"Welcome back. The gate is open."* → Exit (`Completed`).

### 6. Wire Up the Rejection Path

BranchPoint[1] output pin → SayLine *"Halt! Who goes there?"* → PlayerChoice with two choices:
- Choice 1: *"I have no pass."* → Exit (`Failed`).
- Choice 2: *"I'm looking for the gatekeeper."* → SayLine *"Then go to the chamber on the left."* → Exit (`Completed`).

### 7. Compile and Test

In the Preview Runner, the dialogue runs with the fallback path (no tag set). Add the tag to the Player ASC in the Debugger to test the success path.

## BranchPoint Evaluation Logic

BranchPoints are evaluated **in order**. The first one whose Requirements return `Passed` wins.

| Result | Behavior |
|----------|-----------|
| `Passed` | This path is taken, the rest is ignored. |
| `FailedButVisible` | Skip, continue to the next point. |
| `FailedAndHidden` | Skip, continue to the next point. |

The last BranchPoint without Requirements is the **default path**. Without a fallback and without a match → the dialogue aborts.

## Setting the GAS Tag for Testing

```cpp
// Somewhere in the quest system, when the step is completed:
UAbilitySystemComponent* ASC = UAbilitySystemBlueprintLibrary::GetAbilitySystemComponent(PlayerPawn);
ASC->AddLooseGameplayTag(FGameplayTag::RequestGameplayTag(TEXT("Story.Pass.CityGate")));
```

{% hint style="info" %}
For **persistent** tags (that should survive a save file) use an Infinite Duration GameplayEffect with `GrantedTags`. `AddLooseGameplayTag` is not automatically saved.
{% endhint %}

## Blueprint Triggering

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset:      DA_Guard_Gate
   ├─ Instigator: Get Player Pawn
   └─ Target:     Self
```

> 📸 **Image placeholder:** `branching-conditions-bp-trigger.png` — Blueprint trigger on the NPC.
> *Setup:* Guard actor BP. Event graph: `Event OnInteract` → `Start Dialogue`. Pins: `Asset = DA_Guard_Gate`, `Instigator = Get Player Pawn (0)`, `Target = Self`.

{% hint style="info" %}
**C++ variant**

```cpp
void AGuardNPC::BeginConversation(AActor* Player)
{
    if (UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(GetWorld()))
    {
        Sub->StartDialogue(GuardAsset, Player, this);
    }
}
```
{% endhint %}

## Variations / Going Further

- Attach a Requirement to a **PlayerChoice** instead of a Branch → hide only a single option. See [Choice Visible Only with Tag](choice-with-tag-requirement.md).
- Check an attribute instead of a tag (e.g. Stamina ≥ 50) → [Choice with Attribute Condition](choice-with-attribute-requirement.md).
- Extract the rejection path as its own asset and reference it via Link → [Reusable Dialogue Fragments](linking-dialogues.md).

## Troubleshooting

**Dialogue always takes the fallback path.**
Open the [Debugger](../editor/debugger.md) in PIE and check whether the tag is actually on the player's ASC. `Story.Pass.CityGate` ≠ `Story.Pass.City.Gate` — exact spelling required. Also: `bCheckOnInstigator` must be `true`, otherwise the NPC's ASC is checked.

**Success path never reached, even though the tag is present.**
`bCheckOnInstigator = false` → the Requirement checks the NPC instead of the player. Or the actor has no ASC. The Requirement then always returns `Fail`.

**Branch aborts with "No BranchPoint passed".**
The last BranchPoint has Requirements. Add an empty default point or set `FailBehavior = Skip` on the Branch node.
