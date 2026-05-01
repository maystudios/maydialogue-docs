---
description: The five widget classes at a glance, how they work together, and how to swap individual components for your own.
---

# UMG Architecture

The UMG UI is **component-based**. Each widget class has a clearly defined responsibility. You subclass exactly the parts you want to customize — leaving the rest unchanged.

> 📸 **Image placeholder:** `umg-component-diagram.png` — Diagram of the component relationship. White background. Top element: rectangle "UMayDialogueWidget (Top-Level)" with dashed border. Below it, connected by an arrow: rectangle "DialogFrame". From DialogFrame, four arrows lead to: "Speaker" (left), "Text" (center-left), "ChoiceList" (center-right), "SkipButton" (right). From ChoiceList, another arrow leads to "ChoiceButton (n×)". All connections labeled "BindWidget". Layout horizontal, minimalist.
> *Setup:* Diagram as vector graphic or simple Blueprint widget hierarchy screenshot from the UMG Designer.

## The Classes at a Glance

| Class | Purpose |
|---|---|
| `UMayDialogueWidget` | Top-Level. Binds to Instance/Participant, distributes events. |
| `UMayDialogueWidget_DialogFrame` | Container. Holds sub-widgets, controls Open/Close animations. |
| `UMayDialogueWidget_Speaker` | Name + portrait + emotion tags. |
| `UMayDialogueWidget_Text` | Typewriter text with rich-text decorators. |
| `UMayDialogueWidget_ChoiceList` | Spawns and manages ChoiceButtons. |
| `UMayDialogueWidget_ChoiceButton` | Individual answer button. |
| `UMayDialogueWidget_SkipButton` | Advance prompt, platform-aware. |

All classes are `Abstract` and `Blueprintable`. You always work with Blueprint subclasses.

## Data Flow

```
Instance/Participant
    │  OnMessageReceived
    ▼
UMayDialogueWidget (Top-Level)
    ├──► DialogFrame → OnDialogueStarted / OnDialogueEnded
    ├──► Speaker     → SetSpeakerData(Name, Portrait, EmotionTags)
    ├──► Text        → StartTypewriter(Text, CPS)
    │         └──► OnCharacterRevealed (hook for Babel synthesis)
    ├──► ChoiceList  → SetChoices(Entries)
    │         └──► OnChoiceSelected → RequestSelectChoice(Index)
    └──► SkipButton  → OnSkip → RequestAdvance()
```

## Two Configuration Paths

### Path A — A Single Composite Blueprint Widget (recommended)

You build a single top-level Blueprint `WBP_MayDialogue` based on `UMayDialogueWidget`. Inside it, you nest the sub-widgets directly:

```
WBP_MayDialogue (Parent: UMayDialogueWidget)
└── Canvas Panel
    └── WBP_DialogFrame (Name: "DialogFrameWidget")
        ├── WBP_Speaker    (Name: "SpeakerWidget")
        ├── WBP_Text       (Name: "TextWidget")
        ├── WBP_ChoiceList (Name: "ChoiceListWidget")
        └── WBP_SkipButton (Name: "SkipButtonWidget")
```

The names of the child widgets must exactly match the `BindWidget` property names. UMG wires them automatically.

Then in the Project Settings:

```ini
[MayDialogue]
DefaultDialogueWidgetClass = /Game/UI/WBP_MayDialogue.WBP_MayDialogue_C
```

> 📸 **Image placeholder:** `umg-designer-hierarchy.png` — UMG Designer, Hierarchy panel on the left. Visible: `WBP_MayDialogue` as root, below it indented `WBP_DialogFrame` (name in "DialogFrameWidget" field), below that `WBP_Speaker` (name "SpeakerWidget"), `WBP_Text` (name "TextWidget"), `WBP_ChoiceList` (name "ChoiceListWidget"), `WBP_SkipButton` (name "SkipButtonWidget"). Red arrow pointing to the name fields.
> *Setup:* Open UMG Designer with WBP_MayDialogue, screenshot the Hierarchy panel.

### Path B — Per-Class Defaults in the Settings

You specify a class for each slot in the Project Settings:

```ini
DefaultSpeakerWidgetClass  = /Game/UI/WBP_MySpeaker.WBP_MySpeaker_C
DefaultTextWidgetClass     = /Game/UI/WBP_MyText.WBP_MyText_C
DefaultChoiceListWidgetClass = ...
```

The system combines them at runtime. Useful when you need different widget combinations per scene type.

Path A is clearer for most projects. Path B is more flexible with multiple theme variants.

## How Do I Swap a Single Component?

Using the Speaker widget as an example — the process is the same for all components:

**Step 1 — Create a Blueprint subclass**

`Content Browser → Right-click → Blueprint Class → Parent: MayDialogueWidget_Speaker`
Name it e.g. `WBP_MySpeaker`.

> 📸 **Image placeholder:** `umg-new-blueprint-speaker.png` — "Pick Parent Class" dialog in UE5. The search bar shows "MayDialogueWidget_Speaker", the match is highlighted. Red arrow on the entry.
> *Setup:* Content Browser → Right-click → Blueprint Class → search field filled with "MayDialogueWidget_Speaker".

**Step 2 — Build the layout**

In the UMG Designer of `WBP_MySpeaker`:
- Add an `Image` Node, name it **`PortraitImage`** (exact name for BindWidget).
- Add a `Text Block`, name it **`NameText`**.
- Add any other elements as desired (emotion icons, background frame, etc.).

**Step 3 — Implement On Speaker Changed**

In the Event Graph: override `On Speaker Changed`.

```text
Event On Speaker Changed (DisplayName, Portrait, EmotionTags)
  → Set Text (NameText, DisplayName)
  → Set Brush from Texture (PortraitImage, Portrait)
  → Branch: EmotionTags contains "Dialogue.Emotion.Scared"
      True  → Set Portrait to P_Scared, Play Anim "Shake"
      False → Branch: contains "Dialogue.Emotion.Angry"
                  True  → Set Portrait to P_Angry, Set Text Color Red
                  False → Set Portrait to P_Neutral
```

> 📸 **Image placeholder:** `umg-speaker-event-graph.png` — Blueprint graph of WBP_MySpeaker. Event "On Speaker Changed" (purple event node) connected to "Set Text" node (NameText target), then "Set Brush From Texture" node (PortraitImage target), then "Branch" node (Condition: EmotionTags Contains Tag). True/False pins lead to further Set-Portrait nodes.
> *Setup:* WBP_MySpeaker → Event Graph → implement On Speaker Changed as described. Screenshot the graph.

**Step 4 — Plug it in**

In `WBP_MayDialogue` (or your DialogFrame): replace the existing Speaker widget child with `WBP_MySpeaker`. The slot name stays `SpeakerWidget`.

{% hint style="success" %}
That's it. The system automatically calls `SetSpeakerData` on your new class — no further wiring needed.
{% endhint %}

## Common Errors

| Problem | Solution |
|---|---|
| SkipButton does not respond | Child widget must be named exactly `SkipButtonWidget`. |
| Portrait does not appear | Portrait is `TSoftObjectPtr` — async load. Wait for `OnSpeakerChanged`. |
| Typewriter does not call Babel | In the component path, bind manually: `TextWidget → OnCharacterRevealed → BabelSynth`. |
| Sub-widget slot is null | Widget name in Designer does not match the property name. |

Next: [Dialog Frame](dialog-frame.md).
