---
description: Setting emotion tags on SayLines and how UI, audio, and animation react to them.
---

# Emotions & Tags

MayDialogue uses **GameplayTag containers** instead of simple enum fields to mark emotions, intensities, and scene context on spoken lines. This chapter shows how to set tags and how UI, audio, and animation react to them.

## What are emotion tags?

Every SayLine node carries an `EmotionTags` field — an `FGameplayTagContainer` into which you enter any number of tags. These tags are **not an enum** — they have a hierarchy and can be combined.

Example tags:

```text
Dialogue.Emotion.Scared
Dialogue.Emotion.Angry
Dialogue.Emotion.Whisper
Dialogue.Intensity.Low
Dialogue.Intensity.High
Dialogue.Scene.CellarWithMonster
Dialogue.Style.InnerMonologue
```

A SayLine can carry multiple tags at once:

```text
Tags: { Dialogue.Emotion.Scared, Dialogue.Intensity.High, Dialogue.Scene.CellarWithMonster }
```

> 📸 **Image placeholder:** `sayline-emotion-tags-details.png` — Details panel of a SayLine with EmotionTags set.
> *Setup:* SayLine node selected in the graph. Details panel on the right shows under "Emotion Tags": three entries: `Dialogue.Emotion.Scared`, `Dialogue.Intensity.High`, `Dialogue.Scene.CellarWithMonster`. In the graph node: the three tags visible as blue chips below the inline text.

## Why tag containers instead of enums?

| Aspect | Tag container | Simple enum |
| --- | --- | --- |
| Multiple values at once | Yes | No |
| Hierarchically matchable | Yes | No |
| Extensible after the fact | Yes, without schema changes | No |
| Auto-complete in the editor | Yes | Yes |

The most important advantage is **hierarchy matching**. A widget can check: "Does this SayLine have any emotion tag?" — without knowing every individual tag:

```cpp
if (Message.EmotionTags.HasTag(FGameplayTag::RequestGameplayTag("Dialogue.Emotion")))
{
    // Found some emotion tag — for example for a portrait swap
}
```

A newly added `Dialogue.Emotion.Guilty` matches automatically, without any code change.

## Setting tags on a SayLine

**In the graph:**

> 📸 **Image placeholder:** `sayline-add-emotion-tag.png` — Adding an emotion tag to a SayLine.
> *Setup:* SayLine node selected in the graph. Details panel, field `EmotionTags`, `+` button clicked. Dropdown with tag picker opens. Entered: `Dialogue.Emotion.Scared`. The tag then appears as a blue chip in the node body.

Tags can be read directly in the graph node as colored chips — without opening the Details panel.

## Who reacts to tags?

### UI: Portrait variants

The widget receives `Message.EmotionTags` with every new SayLine. In the Blueprint event `OnMessageReceived` you can swap the portrait:

> 📸 **Image placeholder:** `bp-portrait-switch-by-tag.png` — Blueprint graph in the widget: swap portrait based on emotion tags.
> *Setup:* Widget Blueprint, event `On Message Received`. `Message.EmotionTags` → `HasTag (Dialogue.Emotion.Scared)` → Branch. True branch: `Set Brush from Texture (TX_Guard_Scared)`. False branch: another Branch with `HasTag (Dialogue.Emotion.Angry)` → `TX_Guard_Angry` / `TX_Guard_Neutral`. All nodes labeled, pins visible.

```cpp
void UMyDialogueWidget::OnMessageReceived(const FMayDialogueMessage& Message)
{
    if (Message.EmotionTags.HasTag(TAG_Emotion_Scared))
        PortraitImage->SetBrushFromTexture(TX_Guard_Scared);
    else if (Message.EmotionTags.HasTag(TAG_Emotion_Angry))
        PortraitImage->SetBrushFromTexture(TX_Guard_Angry);
    else
        PortraitImage->SetBrushFromTexture(TX_Guard_Neutral);
}
```

### Audio: Intensity and mood

An audio manager listens to `OnMessageReceived` and adjusts the mixer or BabelProfile:

```cpp
void UAudioManager::HandleMessage(const FMayDialogueMessage& Message)
{
    if (Message.EmotionTags.HasTag(TAG_Intensity_High))
        SetMixerSnapshot("DialogueTense");
    else
        SetMixerSnapshot("DialogueCalm");
}
```

### Animation: Face blending

A face blend controller can drive morph targets based on emotion tags — without the dialogue node having a direct animation dependency. The controller listens to `OnMessageReceived` and blends independently from the dialogue.

> 📸 **Image placeholder:** `ingame-emotion-portrait.png` — In-game dialogue with a swapped portrait.
> *Setup:* PIE mode, active dialogue. The widget shows the frightened guard portrait (TX_Guard_Scared) with red tinting. Below it the text "Th-there's... something in the cellar." (typewriter running). No choices visible. Background: cellar level.

## Tags on choices

Choices can additionally carry their own `ChoiceTags`:

```text
Choice 0: "I know the password."
  Tags: { Dialogue.Choice.Assertive, Dialogue.Choice.Secret }

Choice 1: "Hello."
  Tags: { Dialogue.Choice.Friendly }
```

`OnChoiceMade` passes the tags of the selected choice — useful for achievements or analytics:

```cpp
void AMyTracker::OnChoiceMade(int32 Index, const FMayDialogueChoiceEntry& Choice)
{
    if (Choice.ChoiceTags.HasTag(TAG_Choice_Assertive))
        Achievements->Grant("AssertivePlayer");
}
```

## Creating your own tag taxonomy

Tags are registered via INI or natively in C++.

**Via INI** (`Config/Tags/DialogueTags.ini`):

```ini
[/Script/GameplayTags.GameplayTagsList]
+GameplayTagList=(Tag="Dialogue.Emotion.Scared",DevComment="NPC is frightened.")
+GameplayTagList=(Tag="Dialogue.Intensity.High",DevComment="Intense scene.")
```

**Natively (C++):**

```cpp
// Header
UE_DECLARE_GAMEPLAY_TAG_EXTERN(TAG_Dialogue_Emotion_Scared)

// Cpp
UE_DEFINE_GAMEPLAY_TAG_COMMENT(TAG_Dialogue_Emotion_Scared,
    "Dialogue.Emotion.Scared", "NPC is frightened.")
```

Native tags have the advantage of typed references in code — no string typos possible.

## Recommended tag categories

| Category | Purpose | Examples |
| --- | --- | --- |
| `Dialogue.Emotion.*` | Core feeling of the speaker | Calm, Scared, Angry, Sad, Happy, Whisper |
| `Dialogue.Intensity.*` | Strength of the emotion | Low, Medium, High |
| `Dialogue.Scene.*` | Spatial context | Kitchen, Office, Graveyard, CellarWithMonster |
| `Dialogue.Style.*` | Narrative style | InnerMonologue, Narrator, Shout |
| `Dialogue.Mood.*` | Relationship tone | Friendly, Neutral, Hostile |
| `Dialogue.Choice.*` | Marking selected responses | Assertive, Friendly, Aggressive |

{% hint style="info" %}
**Custom tag categories** can be added at any time — simply register them in the INI or via `UE_DEFINE_GAMEPLAY_TAG_COMMENT`. Existing code matches automatically via hierarchy, without any changes.
{% endhint %}

> 📸 **Image placeholder:** `tag-picker-in-editor.png` — Tag picker in the dialogue asset with custom Dialogue tags.
> *Setup:* SayLine node selected, tag picker dropdown for `EmotionTags` open. Visible: tree structure with `Dialogue > Emotion` expanded: `Calm`, `Scared`, `Angry`, `Sad`, `Whisper`. Below it `Dialogue > Intensity`: `Low`, `Medium`, `High`. One tag is currently highlighted on hover.

## Summary

- Emotion tags are `GameplayTagContainer` — hierarchical, combinable, extensible without schema changes.
- Set tags: in the SayLine's Details panel or directly via double-click.
- Consumers: UI (portrait variants), audio (mixer/Babel), animation (face blending).
- Choice tags on `FMayDialogueChoiceEntry` for achievements and analytics.
- `OnMessageReceived` and `OnChoiceMade` are the hooks for external systems.

End of Core Concepts. Next: the **Editor**: [Asset Editor](../editor/README.md).
