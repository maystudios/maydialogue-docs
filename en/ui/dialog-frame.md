---
description: The Dialog Frame as a container widget — what it must contain, the interface it provides, and how to add animations.
---

# Dialog Frame

`UMayDialogueWidget_DialogFrame` is the container that wraps all other dialogue widgets. It handles layout, background, and Open/Close animations. All sub-widgets are optional — you include exactly the ones you need.

> 📸 **Image placeholder:** `dialog-frame-ingame.png` — PIE viewport, running dialogue. The Frame widget is visible: background panel at the bottom, inside it Speaker at top left, text in the center, Choice buttons below, Skip hint at bottom right. Red border around the entire Frame widget.
> *Setup:* Start PIE, trigger dialogue. Add annotation border around the Frame area.

## What the Frame Must Contain

The Frame holds four optional sub-widget slots via `BindWidget`. The names in the UMG Designer must match exactly:

| Property Name | Type | Purpose |
|---|---|---|
| `SpeakerWidget` | `UMayDialogueWidget_Speaker` | Name + portrait |
| `TextWidget` | `UMayDialogueWidget_Text` | Typewriter text |
| `ChoiceListWidget` | `UMayDialogueWidget_ChoiceList` | Answer buttons |
| `SkipWidget` | `UMayDialogueWidget_SkipButton` | Advance prompt |

> 📸 **Image placeholder:** `dialog-frame-umg-hierarchy.png` — UMG Designer, Hierarchy panel. WBP_DialogFrame as root. Below it: Canvas Panel, inside it four child widgets with their exact property names visible. Details panel on the right shows the name of the highlighted slot.
> *Setup:* Open WBP_DialogFrame in the UMG Designer, screenshot the Hierarchy panel. Mark property names with a red arrow.

## Interface

### Blueprint Events — Control Animations Here

```cpp
// Called when a dialogue starts. Play your intro animation here.
void OnDialogueStarted(const FMayDialogueMessage& Message)

// Called when a dialogue ends. Play your outro animation here.
void OnDialogueEnded()
```

### Callable Functions

```cpp
void ShowFrame()   // Default: Visibility = SelfHitTestInvisible
void HideFrame()   // Default: Visibility = Collapsed
```

Override these in your Blueprint subclass to attach animations.

## Adding Animations — Step by Step

**Step 1 — Create UMG Animation**

In the UMG Designer → Animations tab → `+` → Name: `FrameIntro`. Set keyframes for Opacity (0→1) and Position (SlideUp, e.g. Translation Y: 50→0), duration 0.3 s.

> 📸 **Image placeholder:** `dialog-frame-animation-panel.png` — UMG Designer, Animations tab at bottom. Animation "FrameIntro" is selected, timeline shows two tracks: Opacity (0.0 → 1.0) and Translation (Y: 50 → 0). Duration 0.3 s.
> *Setup:* Open UMG Designer, show Animations tab, create FrameIntro animation and screenshot.

**Step 2 — Implement On Dialogue Started**

```text
Event On Dialogue Started (Message)
  → Set Visibility (Self, SelfHitTestInvisible)
  → Play Animation (FrameIntro, NumLoopsToPlay: 1, PlayMode: Forward)
```

**Step 3 — Implement On Dialogue Ended**

```text
Event On Dialogue Ended
  → Play Animation (FrameOutro, NumLoopsToPlay: 1, PlayMode: Forward)
  → Delay (outro duration, e.g. 0.25 s)
  → Set Visibility (Self, Collapsed)
```

> 📸 **Image placeholder:** `dialog-frame-event-graph.png` — Blueprint graph of WBP_DialogFrame. Two event chains side by side: "On Dialogue Started" → Set Visibility → Play Animation "FrameIntro". "On Dialogue Ended" → Play Animation "FrameOutro" → Delay → Set Visibility Collapsed.
> *Setup:* Open WBP_DialogFrame Event Graph, implement both events, screenshot the graph.

## Typical Layout Pattern

```
WBP_DialogFrame
└── Canvas Panel
    ├── Background (Image / Blur)        ← Theme: dark panel, parchment, etc.
    ├── SpeakerWidget  (top left)
    ├── TextWidget     (center)
    ├── ChoiceListWidget (bottom)
    └── SkipWidget     (bottom right)
```

## Theme Hooks

The Frame is the right place for all visual theme decisions:

- **Background image** — Parchment, dark surface, or transparent box.
- **Border style** — Sharp corners for horror, soft rounded for VN.
- **Color scheme** — Via widget animations or material parameters.

See [Themes & Starter Kits](themes.md) for ready-made example setups.

{% hint style="info" %}
**Custom variant:** Create a Blueprint subclass of `UMayDialogueWidget_DialogFrame`. Override `ShowFrame`/`HideFrame` for custom animation logic. Implement `OnDialogueStarted`/`OnDialogueEnded` for lifecycle reactions.
{% endhint %}

{% hint style="warning" %}
The Frame does **not technically own** its sub-widgets — it holds references. The sub-widgets are children of the top-level widget. Changes to the Frame's visibility do not automatically affect sub-widgets.
{% endhint %}
