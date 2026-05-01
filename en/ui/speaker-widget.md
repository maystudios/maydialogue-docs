---
description: The Speaker widget — what data it receives, how to bind portrait and emotion tags, and how to replace it entirely.
---

# Speaker Widget

`UMayDialogueWidget_Speaker` displays the name, portrait, and emotion of the current Speaker. It receives all data through a single call — `SetSpeakerData` — and then fires the Blueprint Event `On Speaker Changed`, in which you control the appearance.

> 📸 **Image placeholder:** `speaker-widget-ingame.png` — PIE viewport, dialogue active. The Speaker widget is visible with portrait on the left (circularly cropped, 128×128), Speaker name bold on the right, name color matching the Speaker color from the graph. Red border around the Speaker area.
> *Setup:* Start PIE, trigger a dialogue with a Speaker who has a portrait and NodeColor set.

## What Data Does the Widget Receive?

The top-level widget calls on every Speaker change:

```cpp
void SetSpeakerData(FText DisplayName, UTexture2D* Portrait, FGameplayTagContainer EmotionTags)
```

Immediately afterward, the Blueprint Event `On Speaker Changed` fires with the same parameters.

### Automatically Wired Slots

If you create these widgets in the UMG Designer with exact names, they are automatically filled:

| Name in Designer | Type | Filled with |
|---|---|---|
| `NameText` | `UTextBlock` | `DisplayName` |
| `PortraitImage` | `UImage` | Portrait texture |

Both slots are optional. Without them you must set everything yourself in the `On Speaker Changed` Event.

## Using Emotion Tags

`EmotionTags` is an `FGameplayTagContainer` — you check it in the Event and react with portrait swaps, animations, or color changes.

```text
Event On Speaker Changed (DisplayName, Portrait, EmotionTags)
  → Branch: EmotionTags Contains Tag "Dialogue.Emotion.Scared"
      True  → Set Portrait Brush = P_Speaker_Scared
               Play Animation "Shake"
      False → Branch: Contains "Dialogue.Emotion.Angry"
                  True  → Set Portrait Brush = P_Speaker_Angry
                           Set NameText Color = (1.0, 0.2, 0.2, 1.0)
                  False → Set Portrait Brush = P_Speaker_Neutral
                           Set NameText Color = White
```

> 📸 **Image placeholder:** `speaker-emotion-graph.png` — Blueprint graph. Event "On Speaker Changed" → Branch chain for EmotionTags. Each Branch path sets a different Portrait Brush and color. All Nodes visible, pin connections clear.
> *Setup:* WBP_MySpeaker → Event Graph → On Speaker Changed. Implement Branch chain, screenshot.

## Portrait Source

The portrait comes from two sources, which the top-level widget resolves automatically:

1. `FMayDialogueSpeaker::Portrait` — portrait on the Speaker asset in the editor.
2. Fallback: `UMayDialogueParticipant::Portrait` on the Participant component of the actor.

The widget always receives a finished `UTexture2D*` — you do not need to manage async loading yourself.

{% hint style="warning" %}
Portraits are loaded synchronously on cold loads — brief hitches are possible. Workaround: preload portraits via `AsyncLoad` at level start.
{% endhint %}

## Using the Speaker Color from the Graph

Each Speaker in the MayDialogue editor has a `NodeColor`. This color is available to you in the widget as `FMayDialogueSpeaker::NodeColor`. Use it to automatically color the Speaker name or accent bar in the graph color — consistent color coding between editor and game.

## Building Your Own Speaker Widget

**Step 1** — Create a Blueprint subclass: Parent `MayDialogueWidget_Speaker`, name e.g. `WBP_MySpeaker`.

**Step 2** — UMG Designer: Image `PortraitImage`, TextBlock `NameText`, plus any custom elements (emotion icon row, background frame, name badge).

> 📸 **Image placeholder:** `speaker-umg-designer.png` — UMG Designer of WBP_MySpeaker. Hierarchy panel: Canvas Panel → Horizontal Box → Image (Name: "PortraitImage") + Vertical Box → TextBlock (Name: "NameText") + Emotion Icon Row. Designer preview shows layout.
> *Setup:* Open WBP_MySpeaker in the UMG Designer, screenshot Hierarchy and Viewport preview.

**Step 3** — Implement `On Speaker Changed` in the Event Graph (see above).

**Step 4** — In `WBP_DialogFrame`, replace the old Speaker child widget with `WBP_MySpeaker`. The slot name stays `SpeakerWidget`.

{% hint style="success" %}
The system automatically calls `SetSpeakerData` on your class. No further wiring needed.
{% endhint %}
