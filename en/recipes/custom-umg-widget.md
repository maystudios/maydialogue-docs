---
description: Connect your own UMG widget as the dialogue UI — replacing the Slate debug widget with a production-ready design.
---

# Connecting a Custom UMG Widget

## Scenario

The included Slate debug widget is fine for prototypes, but your horror game needs a tailor-made dialogue UI: a dirty typeface, CRT scanlines, a hand-drawn frame. This recipe shows you how to register your own UMG widget as the dialogue frame and bind the player data.

## What you will learn

- Creating your own UMG widget as a subclass of the MayDialogue frame widget.
- Registering the widget in Project Settings.
- Required bindings: SpeakerName, DialogueText, ChoiceList.
- Binding the typewriter engine to your own TextBlock.
- Countdown binding for timed choices.

## Prerequisites

- [Quick Start](../getting-started/quick-start.md) completed.
- Basic UMG knowledge (UserWidget, TextBlock, ListView).

## What you will create

A widget `WBP_DialogFrame_Horror` with:
- TextBlock for the speaker name.
- RichTextBlock for dialogue text (typewriter-ready).
- ListView or VerticalBox for the choice buttons.
- ProgressBar for the countdown timer.

> 📸 **Image placeholder:** `custom-umg-widget-final-design.png` — Finished horror dialogue widget in the UMG Editor.
> *Setup:* Widget `WBP_DialogFrame_Horror` open in the UMG Editor. Visible: dark background with vignette, speaker name in the top-left corner (white, pixel font), large RichTextBlock in the centre, choice buttons at the bottom (3 buttons, VHS style). ProgressBar top-right for countdown.

## Step by step

### 1. Create the widget base class

New Widget Blueprint: Parent = **`UMayDialogueFrameWidget`** (MayDialogue base class).

Name: `WBP_DialogFrame_Horror`.

{% hint style="info" %}
**Building your own variant:** `UMayDialogueFrameWidget` is the base class for the dialogue frame widget. It declares the required interface functions you must override in the subclass.
{% endhint %}

### 2. Build the UMG layout

Add the following elements in the Designer panel:

| Element | Name | Purpose |
|---------|------|---------|
| TextBlock | `SpeakerNameText` | Speaker display name |
| RichTextBlock | `DialogueBodyText` | Line text with typewriter |
| VerticalBox or ListView | `ChoiceContainer` | Choice buttons |
| ProgressBar | `TimeoutBar` | Countdown progress |
| Button sub-widget | `WBP_ChoiceButton` | One per choice |

> 📸 **Image placeholder:** `custom-umg-widget-hierarchy.png` — Widget hierarchy in the UMG Designer.
> *Setup:* UMG Designer, left panel shows hierarchy: Canvas → Overlay → VBox → [SpeakerNameText, DialogueBodyText, ChoiceContainer (VerticalBox), TimeoutBar]. All elements with correct names in the list.

### 3. Override required functions

Implement the following functions (Override) in the widget's Event Graph:

**`OnSpeakerChanged(SpeakerData)`:**
```text
[SpeakerData → DisplayName] → [SpeakerNameText → SetText]
```

**`OnLineChanged(LineData)`:**
```text
[LineData → DialogueText] → [MayTypewriterComponent → StartTypewriter(DialogueBodyText)]
```

**`OnChoicesUpdated(Choices)`:**
```text
[Clear ChoiceContainer]
[ForEach Choice: Create WBP_ChoiceButton → Add to ChoiceContainer]
```

**`OnCountdownTick(Progress)`** (for timed choice):
```text
[Progress] → [TimeoutBar → SetPercent]
```

> 📸 **Image placeholder:** `custom-umg-widget-event-graph.png` — Event Graph with the OnLineChanged implementation.
> *Setup:* WBP_DialogFrame_Horror Event Graph. Override function `OnLineChanged` expanded. Nodes: `LineData → Break FMayDialogueLineData` → `DialogueText` pin → `MayTypewriterComponent → StartTypewriter` → `DialogueBodyText`. Typewriter component visible top-left in the Components panel.

### 4. Create the choice button widget

New widget `WBP_ChoiceButton`. Contains:
- Button with TextBlock for the choice text.
- Optional lock icon for requirements.
- `OnClicked` → `MayDialogueLibrary → Submit Choice`.

```text
[Button OnClicked]
   │
   ▼
[MayDialogueLibrary → Submit Choice]
   ├─ ChoiceIndex: (variable, set when the button is created)
   └─ Context:     (from the parent widget)
```

### 5. Register the widget in Project Settings

**Edit → Project Settings → MayDialogue → UI**:
- `DialogFrameWidgetClass` = `WBP_DialogFrame_Horror`.

From now on MayDialogue starts this widget instead of the Slate debug widget.

### 6. Bind the typewriter

In the widget: **Add Component → MayTypewriterComponent**. In `OnLineChanged`:
```text
[MayTypewriterComponent → StartTypewriter]
   ├─ TargetRichTextBlock: DialogueBodyText
   ├─ Text:                LineData.DialogueText
   └─ CharsPerSec:         60
```

The typewriter handles revealing the text character by character. Bind the `OnTypewriterFinished` delegate for advance mode logic.

> 📸 **Image placeholder:** `custom-umg-widget-ingame.png` — Horror dialogue widget in PIE with real content.
> *Setup:* PIE screenshot. Widget `WBP_DialogFrame_Horror` visible over the player. Speaker name `"Guard"` in pixel font. Typewriter running through the text. Two choice buttons at the bottom.

## Variations / going further

- **Keep the Slate debug widget** alongside for quick tests: configurable in the editor settings.
- **Portrait image**: `SpeakerData.Portrait` → `Image → SetBrushFromTexture`.
- **Rich text tags** for coloured keywords in dialogue → [UI → Rich Text Tags](../ui/rich-text-tags.md).

## Troubleshooting

**Widget does not appear.**
`DialogFrameWidgetClass` not set in Project Settings, or wrong widget type (must inherit from `UMayDialogueFrameWidget`).

**Typewriter shows text immediately in full.**
Typewriter component not started. Check `StartTypewriter` in `OnLineChanged`. Or `bSkipTypewriter = true` is enabled in Project Settings.

**Choices are not displayed.**
`OnChoicesUpdated` not implemented correctly, or choice buttons are being added to the wrong container. Check the container name in the Hierarchy panel.
