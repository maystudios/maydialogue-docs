---
description: Check tags, apply effects, count variables – the complete GAS loop in one dialogue.
---

# GAS-Driven Dialogue

## Scenario

An herbalist offers the player a healing potion. She checks their health (attribute), recognizes whether they already know her (tag), heals them on request (GameplayEffect), remembers the potion purchases (participant variable), and grants a story tag on the first meeting. This recipe shows the complete GAS loop: read, write, react.

## What You Will Learn

- Branch on HasTag for first vs. return meeting.
- CheckAttribute Requirement on a choice.
- ApplyEffect action node for a healing potion.
- SideEffect sub-node for a counter increment.
- Set AddTag permanently via the GAS variant.

## Prerequisites

- [Branching with Conditions](branching-conditions.md) completed.
- GAS active, AttributeSet with `Health` registered.

## Mini-Graph

```text
[Entry]
   │
   ▼
[Branch]
   ├─ BP1: HasTag(Story.Met.Herbwoman) → [SayLine: "Good to see you again."]
   └─ BP2: <Fallback>                  → [SayLine: "I don't know you yet."]
                                               │
                                          [AddTag: Story.Met.Herbwoman → Player]
                                               │
                                        ┌──────┴──────────────────────────┐
                                        ▼
                                  [PlayerChoice]
                                    ├─ "A potion, please."  (Req: Health < 50, FailedButVisible)
                                    │       └─► [ApplyEffect: GE_SmallHeal]
                                    │               ├─ SideEffect: PotionsBought += 1
                                    │               └─► [SayLine: "Better?"] → [Exit: Completed]
                                    └─ "No thanks." → [Exit: Completed]
```

> 📸 **Image placeholder:** `gas-driven-dialogue-graph-overview.png` — Complete herbalist dialogue in the Asset Editor.
> *Setup:* Asset `DA_Herbwoman_Visit` open. Visible: Entry → Branch (diamond, two outputs). Top path: SayLine return meeting → PlayerChoice. Bottom path: SayLine first meeting → AddTag node (light green box) → PlayerChoice. PlayerChoice with two choices, top choice has lock icon (Requirement). From Choice 1: ApplyEffect node (purple box) → SayLine → Exit. From Choice 2: directly to Exit.

## Step by Step

### 1. Define Tags and Variable

In `DefaultGameplayTags.ini`:
```ini
+GameplayTagList=(Tag="Story.Met.Herbwoman",DevComment="Player knows the herbalist")
+GameplayTagList=(Tag="Dialogue.Speaker.Herbwoman",DevComment="")
```

In the asset `DA_Herbwoman_Visit` under **Variables panel**: variable `PotionsBought`, type `Int`, scope `Participant`, default `0`.

### 2. Branch for First/Return Meeting

Insert a Branch node. BranchPoint[0]: HasTag Requirement:

| Property | Value |
|----------|------|
| `RequiredTag` | `Story.Met.Herbwoman` |
| `bCheckOnInstigator` | `true` |

BranchPoint[1]: empty (fallback).

### 3. First Meeting Path: Set the Tag

After the introductory SayLine, an **AddTag** action node:

| Property | Value |
|----------|------|
| `TargetParticipantTag` | `Dialogue.Participant.Player` |
| `Tag` | `Story.Met.Herbwoman` |
| `bApplyPermanent` | `true` (via Infinite GE) |

> 📸 **Image placeholder:** `gas-driven-dialogue-addtag-details.png` — Details panel of the AddTag node.
> *Setup:* AddTag node selected. Details: `TargetParticipantTag = Dialogue.Participant.Player`, `Tag = Story.Met.Herbwoman`, `bApplyPermanent = true`.

### 4. PlayerChoice with Attribute Gate

First choice entry gets a **CheckAttribute** Requirement:

| Property | Value |
|----------|------|
| `Attribute` | `UVHSAttributeSet::Health` |
| `ComparisonOp` | `<` |
| `ComparisonValue` | `50.0` |
| `FailureResult` | `FailedButVisible` |
| `UnavailableReason` | *"You seem healthy."* |

The player always sees the option — but can only select it when Health < 50. On hover, the reason appears.

### 5. ApplyEffect with Counter SideEffect

From the healing choice output → **ApplyEffect** node:

| Property | Value |
|----------|------|
| `EffectClass` | `GE_SmallHeal` |
| `TargetParticipantTag` | `Dialogue.Participant.Player` |
| `EffectLevel` | `1.0` |

On the ApplyEffect node, **SideEffect → SetVariable**:
- `Name: PotionsBought`, `Scope: Participant`, `Op: Increment`, `Value: 1`.

> 📸 **Image placeholder:** `gas-driven-dialogue-applyeffect-details.png` — ApplyEffect node with SideEffect pill.
> *Setup:* ApplyEffect node selected. Details: `EffectClass = GE_SmallHeal`, `TargetParticipantTag = Dialogue.Participant.Player`. In the node body below: SideEffect pill `SetVariable PotionsBought += 1`.

### 6. Compile and PIE Test

In the Debugger after the potion: check that Health rises, `Story.Met.Herbwoman` is on the Player ASC, and `PotionsBought = 1`.

## Loose Tag vs. GameplayEffect

| Mode | Pro | Con |
|-------|-----|--------|
| `LooseGameplayTag` | Simple, no GE needed | Not replicated, not persisted |
| Via Infinite GE | Replicated, persisted | Requires a GE asset |

For story tags that should go into the SaveGame: **always the GE variant**.

## Blueprint Triggering

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset:      DA_Herbwoman_Visit
   ├─ Instigator: Get Player Pawn
   └─ Target:     Self
```

## Variations / Going Further

- Save `PotionsBought` persistently → [Persistence → Participant Memory](../persistence/participant-memory.md).
- Use a CheckAttribute Requirement for an entire Branch decision instead of just one choice.
- Build a custom GAS Requirement (e.g. "player has Ability X") → [Build Custom Requirement in Blueprint](custom-blueprint-requirement.md).

## Troubleshooting

**ApplyEffect has no impact.**
`EffectClass` is empty. Target has no ASC. The GE is Instant but the AttributeSet clamps `Current = Max` in `PostGameplayEffectExecute`.

**Tag set, Branch still takes fallback.**
Loose tag and Branch checked in the same frame. In rare cases MayDialogue caches the tag state — workaround: use the GE variant.

**CheckAttribute never returns Passed.**
Attribute dropdown empty or incorrectly linked. The AttributeSet must be registered on the player's ASC.
