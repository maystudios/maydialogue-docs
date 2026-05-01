---
description: Minimal NPC conversation with two SayLines – the perfect introduction to MayDialogue.
---

# Simple NPC Conversation

## Scenario

A villager greets the player, says two sentences, and the conversation ends. No branching, no conditions, no GAS. This recipe sharpens your feel for how `Entry`, `SayLine`, and `Exit` work together — and which `AdvanceMode` makes sense when.

## What You Will Learn

- Create and name a dialogue asset.
- Fill in the Speakers panel (tag + display name).
- Connect SayLine nodes and set `AdvanceMode`.
- Test the dialogue in the Preview Runner without starting PIE.

## Prerequisites

- [Quick Start](../getting-started/quick-start.md) completed.
- Plugin enabled, widget configured in Project Settings.

## Mini-Graph

```text
[Entry]
   │
   ▼
[SayLine: Villager – "Welcome to Kreuzhof, stranger."  AdvanceMode: Manual]
   │
   ▼
[SayLine: Villager – "Strange things happen at night. Take care of yourself."  AdvanceMode: Manual]
   │
   ▼
[Exit: Completed]
```

> 📸 **Image placeholder:** `simple-npc-talk-graph-overview.png` — Overview of the finished graph in the Asset Editor.
> *Setup:* Asset `DA_Villager_Intro` open. Left to right: green `Entry` node → `SayLine "Welcome to Kreuzhof"` (title bar warm-yellow, speaker: Villager) → `SayLine "Strange things..."` (same color) → red `Exit: Completed` node. Connection lines clearly visible, no comment block.

## Step by Step

### 1. Create the Asset

In the Content Browser: **Right-click → Miscellaneous → May Dialogue Asset**. Name: `DA_Villager_Intro`.

### 2. Register the Speaker

Open **Window → Speakers**. Click **Add Speaker**:

| Field | Value |
|------|------|
| `DisplayName` | Villager |
| `SpeakerTag` | `Dialogue.Speaker.Villager` |
| `Color` | Warm yellow (color picker) |

> 📸 **Image placeholder:** `simple-npc-talk-speakers-panel.png` — Speakers panel with a filled-in entry.
> *Setup:* Speakers panel open (left tab). Single entry: `DisplayName = "Villager"`, `SpeakerTag = Dialogue.Speaker.Villager`, color chip warm-yellow.

### 3. Connect the Nodes

1. Drag a wire from the **Entry** output pin → **Create Node → Say Line**.
2. `SpeakerTag` = `Dialogue.Speaker.Villager`, `DialogueText` = *"Welcome to Kreuzhof, stranger."*, `AdvanceModeOverride` = `Manual`.
3. From the output of this SayLine, create a second SayLine: *"Strange things happen at night. Take care of yourself."*
4. From the second SayLine's output → **Exit** node, `ExitStatus = Completed`.
5. **Compile** (toolbar at the top).

> 📸 **Image placeholder:** `simple-npc-talk-sayline-details.png` — Details panel of the first SayLine.
> *Setup:* First SayLine selected in the graph. Details panel shows: `SpeakerTag = Dialogue.Speaker.Villager`, `DialogueText = "Welcome to Kreuzhof, stranger."`, `AdvanceModeOverride = Manual`. Voice slot empty.

### 4. Test in the Preview Runner

Click **Play** (toolbar) → the dialogue runs directly in the editor. Advance with **Space**, cancel with **Esc**.

## Advance Modes at a Glance

| Mode | Advance triggered by |
|-------|---------------|
| `Manual` | Player input (click / Enter) |
| `Timer` | `AutoAdvanceDelay` elapses |
| `AfterVoice` | Voice asset playback ends |
| `Immediate` | Immediately, without waiting |
| `AfterAnimation` | Montage on the participant ends |

For interactive NPC conversations: **Manual**. For cutscenes without player input: **Timer** or **AfterVoice**.

## Blueprint Triggering

When the player interacts with the NPC (e.g. via an Interaction component):

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset:      DA_Villager_Intro
   ├─ Instigator: Player Pawn
   └─ Target:     Self (the NPC actor)
```

> 📸 **Image placeholder:** `simple-npc-talk-bp-trigger.png` — Blueprint graph of the trigger event.
> *Setup:* Any actor Blueprint open. Event graph shows: `Event OnInteract` → `Start Dialogue (MayDialogueLibrary)` with filled pins: `Asset = DA_Villager_Intro`, `Instigator = Get Player Pawn`, `Target = Self`.

{% hint style="info" %}
**C++ variant**

```cpp
if (UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(GetWorld()))
{
    Sub->StartDialogue(DA_Villager_Intro, PC->GetPawn(), this);
}
```
{% endhint %}

## Variations / Going Further

- Replace a SayLine with a **RandomLine** for varied greetings → [Random Greetings](random-greetings.md).
- Add a **PlayerChoice** node so the player can respond → [Branching with Conditions](branching-conditions.md).
- Extract the SayLines as a separate fragment → [Reusable Dialogue Fragments](linking-dialogues.md).

## Troubleshooting

**Dialogue ends immediately after the first line.**
Check: the output pin of SayLine 1 points directly to `Exit` instead of SayLine 2. Or `AdvanceModeOverride = Immediate` is set — switch to `Manual`.

**Player input doesn't trigger advance.**
In [Project Settings](../getting-started/project-settings.md), `bSwitchToUIInputDuringDialogue` must be `true`. The widget also needs to receive focus (`SetKeyboardFocus` in `NativeConstruct`).

**Name "Villager" appears empty in the widget.**
Only the tag is set in the Speakers panel, but no `DisplayName`. Enter the display name in the panel.
