---
description: ChoiceList and ChoiceButton — how they work together, what layout options are available, and how to build your own button styles.
---

# Choice List & Choice Button

Two widgets that belong together: **ChoiceList** is the container, **ChoiceButton** is the individual answer button. You replace them separately — custom button style, without touching the ChoiceList.

> 📸 **Image placeholder:** `choice-list-ingame.png` — PIE viewport, PlayerChoice node active. Visible: three choice buttons arranged vertically. Button 1 active (full opacity, hover effect), button 2 active, button 3 greyed out (unavailable). Red border around the entire ChoiceList area.
> *Setup:* Start a dialogue with a PlayerChoice node (3 choices, one with an unmet requirement). PIE screenshot.

## UMayDialogueWidget_ChoiceList

The container spawns and manages all ChoiceButtons. You bind a panel container and specify a button class.

### BindWidget slot

| Name in Designer | Type | Purpose |
|---|---|---|
| `ChoiceContainer` | `UPanelWidget` (e.g. VerticalBox) | Buttons are added here |

### Property

**`ChoiceButtonClass`** — A subclass of `MayDialogueWidget_ChoiceButton` (selectable via Class Picker in the editor). Set this value in the Blueprint defaults of your ChoiceList subclass to use a custom button style.

### Blueprint event

```cpp
// Fires after SetChoices, once all buttons have been built.
void OnChoicesSet(int32 ChoiceCount)
```

Hook for things like scroll reset, setting focus to the first button, and fade-in animations.

### Internal flow

```
SetChoices(Entries)
  → ClearChoices()
  → for each entry: spawn ChoiceButton from ChoiceButtonClass
                    call SetChoiceEntry(Entry)
                    OnClicked → HandleChoiceClicked(Index)
  → fire OnChoicesSet(Count)
```

`OnChoiceSelected` (delegate) → top-level widget → `RequestSelectChoice(Index)`.

## UMayDialogueWidget_ChoiceButton

The individual answer button. Each button knows its index and its availability status.

### BindWidget slots

| Name in Designer | Type | Purpose |
|---|---|---|
| `ClickButton` | `UButton` | The clickable button area |
| `ChoiceTextBlock` | `UTextBlock` | Choice text |

### Blueprint events

```cpp
// Fires when the button is populated with data. Use this for styling based on availability.
void OnChoiceSet(FText ChoiceText, bool bIsAvailable, FText UnavailableReason)

void OnChoiceHovered()
void OnChoiceUnhovered()
```

### Availability status

| Status | Visibility | Clickable |
|---|---|---|
| `Passed` | Visible, full opacity | Yes |
| `FailedButVisible` | Visible, greyed out | No (tooltip with reason) |
| `FailedAndHidden` | Collapsed (no layout space) | No |

```text
Event On Choice Set (ChoiceText, bIsAvailable, UnavailableReason)
  → Set Text (ChoiceTextBlock, ChoiceText)
  → Branch: bIsAvailable
      True  → Set Color Alpha = 1.0, Set Button Enabled = true
      False → Set Color Alpha = 0.4, Set Button Enabled = false
               Set ToolTip = UnavailableReason
```

> 📸 **Image placeholder:** `choice-button-event-graph.png` — Blueprint graph of WBP_MyChoiceButton. Event "On Choice Set" → Set Text → Branch → True path (Alpha 1.0, Enabled true) + False path (Alpha 0.4, Enabled false, set tooltip). All nodes visible.
> *Setup:* WBP_MyChoiceButton → Event Graph → implement On Choice Set. Take a screenshot of the graph.

### Configurable disabled colour

```cpp
// Configurable in Blueprint defaults:
FLinearColor DisabledTextColor = FLinearColor(0.4, 0.4, 0.4, 1.0)
```

Adjust the colour per Blueprint subclass to match your visual theme.

## Layout options

**Vertical (default)**

```
WBP_ChoiceList
└── VerticalBox (Name: "ChoiceContainer")
    ├── WBP_ChoiceButton (Choice 0)
    ├── WBP_ChoiceButton (Choice 1)
    └── WBP_ChoiceButton (Choice 2)
```

**Horizontal / Grid**

Swap `VerticalBox` for `HorizontalBox` or `UniformGridPanel` — the container is `UPanelWidget` and therefore freely replaceable.

**With scrolling**

Wrap in a `ScrollBox` when many choices can exceed the visible area:

```
WBP_ChoiceList
└── ScrollBox
    └── VerticalBox (Name: "ChoiceContainer")
```

> 📸 **Image placeholder:** `choice-list-umg-designer.png` — UMG Designer of WBP_ChoiceList. Hierarchy: Canvas → VerticalBox (Name "ChoiceContainer"). Details panel of the VerticalBox visible. Next to it in the viewport: three placeholder buttons.
> *Setup:* Open WBP_ChoiceList in the UMG Designer, take a screenshot of the hierarchy and viewport preview.

## Building your own ChoiceButton

**Step 1** — Blueprint subclass: Parent `MayDialogueWidget_ChoiceButton`, name e.g. `WBP_MyChoiceButton`.

**Step 2** — UMG Designer: add a `Button` (Name: `ClickButton`) and `TextBlock` (Name: `ChoiceTextBlock`), plus any custom elements (index number, key hint, icon).

**Step 3** — Implement `On Choice Set`, `On Choice Hovered`, `On Choice Unhovered`.

**Step 4** — In the Blueprint subclass of your ChoiceList: set `ChoiceButtonClass = WBP_MyChoiceButton`.

{% hint style="success" %}
You swap the button style without touching the ChoiceList. The ChoiceList automatically spawns your new class.
{% endhint %}
