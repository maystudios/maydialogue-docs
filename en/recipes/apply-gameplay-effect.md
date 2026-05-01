---
description: Apply a GameplayEffect directly from dialogue – buff, debuff, or healing as an action node.
---

# Apply GameplayEffect from Dialogue

## Scenario

A priest heals the player mid-conversation. A witch curses them. A blacksmith grants a temporary damage buff. All of this happens via the **ApplyEffect** action node — a single node in the graph that applies any GameplayEffect to the Instigator or Target. No Blueprint code needed.

## What You Will Learn

- Create and configure an ApplyEffect node.
- Choose the target (Instigator vs. Target Actor).
- Set Effect Level and Context.
- Apply multiple effects in sequence.

## Prerequisites

- GAS active, AttributeSets registered.
- GameplayEffect assets created (e.g. `GE_PriestHeal`, `GE_WitchCurse`).

## Mini-Graph – Healing Dialogue

```text
[Entry]
   │
   ▼
[SayLine: Priest – "May the light heal you."]
   │
   ▼
[ApplyEffect: GE_PriestHeal → Instigator (Player)  Level: 1.0]
   │
   ▼
[SayLine: Priest – "Go in peace."]
   │
   ▼
[Exit: Completed]
```

> 📸 **Image placeholder:** `apply-gameplay-effect-graph-overview.png` — Priest dialogue with ApplyEffect node.
> *Setup:* Asset `DA_Priest_Heal` open. From left: Entry → SayLine (blue, priest) → ApplyEffect node (purple box, effect name visible) → SayLine → Exit. ApplyEffect node prominently in the main flow.

## Step by Step

### 1. Prepare the GameplayEffect Asset

In UE: new Blueprint asset with parent `UGameplayEffect`. For a healing effect: `Instant` Duration, `Attribute: Health`, `Magnitude: 50`. Asset name: `GE_PriestHeal`.

### 2. Create the Dialogue Asset

Asset: `DA_Priest_Heal`. Speaker: `Dialogue.Speaker.Priest`.

### 3. Insert the ApplyEffect Node

From the SayLine output → **Create Node → Apply Effect**.

| Property | Value |
|----------|------|
| `EffectClass` | `GE_PriestHeal` |
| `TargetParticipantTag` | `Dialogue.Participant.Player` |
| `EffectLevel` | `1.0` |
| `bApplyFromInstigator` | `true` |

`bApplyFromInstigator = true` → the player is the Instigator of the effect (Source = player ASC). Important for Magnitude calculators that read from Source attributes.

> 📸 **Image placeholder:** `apply-gameplay-effect-node-details.png` — Details panel of the ApplyEffect node.
> *Setup:* ApplyEffect node selected. Details: `EffectClass = GE_PriestHeal`, `TargetParticipantTag = Dialogue.Participant.Player`, `EffectLevel = 1.0`, `bApplyFromInstigator = true (checkbox)`.

### 4. Compile and Test

In PIE: start the dialogue. After the ApplyEffect node, the player's Health attribute should increase. Verify in the GAS Debugger.

## Applying Effects to the NPC

When the priest buffs themselves or the witch curses the player:

| Scenario | `TargetParticipantTag` | `bApplyFromInstigator` |
|----------|----------------------|-----------------------|
| Player heals player | `Dialogue.Participant.Player` | `true` |
| NPC buffs NPC | `Dialogue.Participant.Priest` | `false` |
| Witch curses player | `Dialogue.Participant.Player` | `false` (Witch = Source) |

## Multiple Effects in Sequence

```text
[ApplyEffect: GE_WitchCurse_DotPoison → Player]
   │
   ▼
[ApplyEffect: GE_WitchCurse_DebuffSpeed → Player]
   │
   ▼
[SayLine: "You will regret this."]
```

Both nodes run in sequence and are visible in the graph as separate steps — easy to read, easy to debug.

## ApplyEffect as a SideEffect

If the effect is a secondary step (e.g. a buff applied in passing when a SayLine is entered):

Add **SideEffect → ApplyEffect** to the SayLine node. The SayLine runs, the effect is applied simultaneously — no separate node in the main flow.

## Blueprint Triggering

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_Priest_Heal
   └─ ...
```

> 📸 **Image placeholder:** `apply-gameplay-effect-ingame.png` — PIE with priest dialogue, Health bar visibly rising.
> *Setup:* PIE running. Dialogue widget shows the priest SayLine. GAS debug overlay (key ` → showdebug abilitysystem) shows Health rising animation after ApplyEffect.

## Variations / Going Further

- Dynamic Effect Level: read `EffectLevel` from a variable (e.g. priest level variable).
- Apply the effect only when Health is below a threshold → Requirement before the ApplyEffect node: [Choice with Attribute Condition](choice-with-attribute-requirement.md).
- Full GAS loop (effect + tag + variable) → [GAS-Driven Dialogue](gas-driven-dialogue.md).

## Troubleshooting

**Effect has no impact.**
`EffectClass` empty — most common mistake. Target has no ASC. Instant GE but `PostGameplayEffectExecute` clamps the attribute.

**EffectLevel has no impact.**
The GameplayEffect uses no Level-dependent Magnitude (ScalableFloat). Level scaling must be explicitly configured in the GE asset.

**Effect applied to the wrong actor.**
`TargetParticipantTag` points to the NPC instead of the player. Check tags in the Speakers/Participants panel.
