---
description: Player inner monologue with 2D audio – no spatial sound, but directly inside the head.
---

# Inner Monologue with 2D Audio

## Scenario

The player finds a diary and reads its contents silently. The voice should not come from the room but feel like a thought — dry, close, without 3D attenuation. You achieve this with a SayLine that uses the player as the speaker and a 2D audio override.

## What You Will Learn

- Configure a SayLine with the player as the speaker.
- Set the 2D audio override on the speaker (no Attenuation asset).
- No NPC as a conversation partner: start the dialogue without a target.
- Adjust the typewriter effect for a thought-like style.

## Prerequisites

- [Simple NPC Conversation](simple-npc-talk.md) completed.
- Voice asset for the inner monologue imported: `SW_InnerVoice_Line01`.

## Mini-Graph

```text
[Entry]
   │
   ▼
[SayLine: Player (Inner Voice) – "That was him. The one everyone talks about."  AdvanceMode: AfterVoice]
   │
   ▼
[SayLine: Player (Inner Voice) – "I should be careful."                         AdvanceMode: Timer 3s]
   │
   ▼
[Exit: Completed]
```

> 📸 **Image placeholder:** `inner-monologue-2d-graph-overview.png` — Graph of the diary monologue.
> *Setup:* Asset `DA_Diary_Page1` open. Entry → two SayLines (title bar dark blue = player speaker) → Exit. In the Speakers panel: speaker "Inner Voice" with 2D override setting visible.

## Step by Step

### 1. Create the "Inner Voice" Speaker

Asset: `DA_Diary_Page1`. **Speakers panel** → **Add Speaker**:

| Field | Value |
|------|------|
| `DisplayName` | *(empty or "Thoughts")* |
| `SpeakerTag` | `Dialogue.Speaker.PlayerInner` |
| `Color` | Dark blue or dark gray |
| `AudioOverride.AttenuationAsset` | *(leave empty = 2D)* |
| `AudioOverride.b2DAudio` | `true` |

The empty `AttenuationAsset` slot combined with `b2DAudio = true` ensures that the voice clip plays as a 2D sound — directly in the mix, without spatial positioning.

> 📸 **Image placeholder:** `inner-monologue-2d-speaker-settings.png` — Speakers panel with 2D audio override.
> *Setup:* Speakers panel. Speaker "Inner Voice": `SpeakerTag = Dialogue.Speaker.PlayerInner`, color chip dark blue, `AudioOverride` expanded: `b2DAudio = true`, `AttenuationAsset = None`.

### 2. Configure the SayLines

First SayLine: `SpeakerTag = Dialogue.Speaker.PlayerInner`, `DialogueText = "That was him..."`, `AdvanceModeOverride = AfterVoice`, Voice asset: `SW_InnerVoice_Line01`.

Second SayLine: `AdvanceModeOverride = Timer`, `AutoAdvanceDelay = 3.0`.

### 3. Adjust the Typewriter Style

For thoughts, a slower typewriter often feels right (more contemplative effect). On the SayLine node under `TypewriterOverride`:
- `CharactersPerSecond = 30` (instead of the default 60).
- `bSkipTypewriterOnAdvance = false` (so the text isn't revealed instantly).

### 4. Start the Dialogue Without a Target

Inner monologues have no NPC as a conversation partner. The start call only passes the Instigator:

```text
[Event OnTriggerEnter (Diary Volume)]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset:      DA_Diary_Page1
   ├─ Instigator: Get Player Pawn
   └─ Target:     (empty / None)
```

> 📸 **Image placeholder:** `inner-monologue-2d-bp-trigger.png` — Blueprint with trigger volume and dialogue start without a target.
> *Setup:* TriggerBox actor. `On Actor Begin Overlap` (with player tag check) → `Start Dialogue`. `Target = None` visible on the pin (unconnected).

{% hint style="info" %}
**C++ variant**

```cpp
// Diary actor, when the player enters the overlap area:
if (UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(GetWorld()))
{
    Sub->StartDialogue(DiaryAsset, PlayerPawn, nullptr); // no target
}
```
{% endhint %}

## DisplayName for the Inner Voice

For a thought style, you often want no name displayed in the widget. Two options:
- Leave `DisplayName` empty in the speaker → widget shows no name.
- In the dialogue widget: set `bHideSpeakerName` for the player speaker (widget configuration).

## Variations / Going Further

- Combine the monologue with a **camera effect**: CameraFocus node on the player or an environmental point → [Camera Pan on Speaker](camera-pan-on-speaker.md).
- **Multiple pages**: separate assets for each diary page + Link nodes for sequential reading.
- Mix: first line 2D (thought), second line 3D (spoken aloud) → two speakers with different audio overrides.

## Troubleshooting

**Voice sounds spatial, not 2D.**
`b2DAudio` not enabled on the speaker, or `AttenuationAsset` accidentally filled. Check the Speakers panel.

**No sound in AfterVoice mode.**
Voice asset not assigned on the SayLine node. `AdvanceMode = AfterVoice` with an empty voice slot falls back to typewriter end — the line continues when the typewriter finishes.

**Dialogue starts, but widget doesn't appear.**
The widget needs an Instigator to know where to render. Check if `Get Player Pawn` returns a valid value.
