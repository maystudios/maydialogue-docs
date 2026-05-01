---
description: PlayerChoice with a countdown timer – when it expires, a default choice is selected automatically.
---

# Timed Choice (Auto-Select)

## Scenario

An enemy gives the player 10 seconds to surrender. If they don't react, the system automatically selects *"I refuse to surrender"* and combat begins. This pattern creates genuine time pressure without a cutscene. The countdown timer runs directly in the dialogue widget.

## What You Will Learn

- Configure a choice timeout on the PlayerChoice node.
- Set the default choice for auto-select.
- The widget shows the countdown automatically.
- Combine with a sound stinger for urgency.

## Prerequisites

- [Simple NPC Conversation](simple-npc-talk.md) completed.

## Mini-Graph

```text
[Entry]
   │
   ▼
[SayLine: Enemy – "You have 10 seconds. Surrender or die."]
   │
   ▼
[PlayerChoice  Timeout: 10s  DefaultChoice: 1]
   ├─ [0] "I surrender."    → [SayLine: "Wise."] → [Exit: Completed]
   └─ [1] "Never!"  (Default) → [SayLine: "Then die."] → [FireEvent: Combat.Start] → [Exit: Failed]
```

> 📸 **Image placeholder:** `timed-choice-graph-overview.png` — PlayerChoice node with timeout setting.
> *Setup:* Asset `DA_Enemy_Ultimatum` open. SayLine → PlayerChoice with two choices. In the Details panel of the PlayerChoice: `Timeout = 10.0`, `DefaultChoiceIndex = 1` (index of the "Never!" choice) visible.

## Step by Step

### 1. Create the PlayerChoice Node

Asset: `DA_Enemy_Ultimatum`. Speaker: `Dialogue.Speaker.Enemy`. SayLine *"You have 10 seconds..."* → **PlayerChoice** node.

### 2. Fill in the Choices

Choice 0 (Index 0): `ChoiceText = "I surrender."`.
Choice 1 (Index 1): `ChoiceText = "Never!"`.

### 3. Configure the Timeout

On the PlayerChoice node in the Details panel:

| Property | Value |
|----------|------|
| `bEnableTimeout` | `true` |
| `TimeoutDuration` | `10.0` |
| `DefaultChoiceIndex` | `1` (index of the "Never!" choice) |

`DefaultChoiceIndex = 1` → when the timer expires, Choice 1 is selected automatically.

> 📸 **Image placeholder:** `timed-choice-player-choice-details.png` — Details panel with timeout settings.
> *Setup:* PlayerChoice node selected. Details: `bEnableTimeout = true`, `TimeoutDuration = 10.0`, `DefaultChoiceIndex = 1`. Below that, the two choices with their texts.

### 4. Wire Up the Outputs

Choice 0 → SayLine *"Wise. You live another day."* → Exit (`Completed`).
Choice 1 → SayLine *"Then die."* → **FireEvent** node (`Combat.Start`) → Exit (`Failed`).

### 5. Countdown in the Widget

The standard dialogue widget displays the countdown timer automatically as soon as `bEnableTimeout = true`. The timer appears as a progress bar or number above the choice list. With a custom UMG widget you need to implement the `CountdownValue` binding yourself (see [Custom UMG Widget](custom-umg-widget.md)).

### 6. Compile and Test in PIE

In PIE: start the dialogue. Wait 10 seconds → Choice 1 is selected automatically, the combat event fires.

## Adding a Tension Sound

For more atmosphere: a PlaySound node before the PlayerChoice:

```text
[SayLine: "You have 10 seconds..."]
   │
   ▼
[PlaySound: SE_TickingClock  Loop: true]
   │
   ▼
[PlayerChoice  Timeout: 10s ...]
```

Stop the looping sound at the Exit node (SideEffect or via Event).

## Blueprint Triggering

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_Enemy_Ultimatum
   └─ ...
```

> 📸 **Image placeholder:** `timed-choice-ingame.png` — Preview Runner with a visible countdown timer.
> *Setup:* Preview Runner open. Choice list shows two choices. Above the choices: countdown bar / number `8` (simulated after 2 seconds). Timer visibly counting down.

## Variations / Going Further

- **Short timeout** (3s) for panic decisions: the player has to react instinctively.
- **Timeout without a visible timer**: `bShowCountdownInUI = false` → the player doesn't know time is running out. Horror pattern.
- Process the timeout result in an external system: `OnDialogueEnded(ExitStatus == Failed)`.

## Troubleshooting

**Timer doesn't run.**
`bEnableTimeout = false` or `TimeoutDuration = 0`. Check both.

**Wrong choice is auto-selected.**
`DefaultChoiceIndex` is 0-based. Index 0 = first choice, Index 1 = second choice. Count the choices in the Details panel.

**Countdown doesn't appear in the widget.**
The standard widget implements the countdown automatically. With a custom UMG widget: read `GetChoiceCountdown()` from the Instance and bind it to the ProgressBar.
