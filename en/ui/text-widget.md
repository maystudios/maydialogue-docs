---
description: The text widget as a typewriter component — binding, rich text decorators, the Babel hook, and skip behaviour.
---

# Text Widget

`UMayDialogueWidget_Text` is the typewriter component. It receives the dialogue text, reveals it character by character, and fires events for Babel synthesis and skip logic.

> 📸 **Image placeholder:** `text-widget-ingame.png` — PIE viewport, dialogue with typewriter running. Visible: partial text (approx. 60% revealed), cursor blinking at the end of the revealed text. Rich text effects active (one word in red, one word with shake). Red border around the text area.
> *Setup:* Start PIE, trigger a dialogue with `<color=red>` and `<shake>` tags in the text, take a screenshot while the typewriter is running.

## Binding in the UMG Designer

The only BindWidget slot:

| Name in Designer | Type | Purpose |
|---|---|---|
| `DialogueRichText` | `URichTextBlock` | Renders the text with all active decorators |

Add a `RichTextBlock` in the UMG Designer and name it **`DialogueRichText`**. The widget populates it automatically.

## Controlling the typewriter

```cpp
// Starts the typewriter. CPS <= 0 uses the Settings default.
void StartTypewriter(FText FullText, float CharactersPerSecond)

// Jumps immediately to the full text.
void SkipTypewriter()

// Resets the text and stops the typewriter.
void ClearText()

// Queries
bool IsTypewriterActive()       // is the typewriter still running?
float GetTypewriterProgress()   // 0.0 = no text, 1.0 = fully revealed
```

## Blueprint events

```cpp
// Fires for each revealed character — hook for Babel synthesis and character SFX.
void OnCharacterRevealed(FString Character, int32 Index)

// Fires when the entire text is revealed (normally or via skip).
void OnTypewriterComplete()
```

## Registering rich text decorators

The `DialogueRichText` widget must know its decorator classes. In the UMG Designer:

1. Select `DialogueRichText`.
2. Details panel → **Decorator Classes** → add entries:
   - `UMayDialogueShakeDecorator`
   - `UMayDialogueWaveDecorator`
   - `UMayDialogueColorDecorator`
   - `UMayDialogueBoldDecorator`

> 📸 **Image placeholder:** `text-widget-decorators.png` — UMG Designer, `DialogueRichText` selected. Details panel on the right shows the "Decorator Classes" section with four entries: Shake, Wave, Color, Bold. Red arrow pointing to this section.
> *Setup:* Open WBP_MyText in the UMG Designer, select DialogueRichText, take a screenshot of the Details panel.

## Binding Babel synthesis

The `OnCharacterRevealed` event is the hook for Babel-style voice effects:

```text
Event On Character Revealed (Character, Index)
  → BabelSynth → OnCharacterRevealed (Character, Index, TotalCharCount)
```

{% hint style="warning" %}
In the component path (sub-widget architecture) the Babel binding is **not automatic**. You bind `OnCharacterRevealed` manually in your Blueprint. Obtain the reference to `BabelSynth` from the top-level widget.
{% endhint %}

## Skip logic

The top-level widget checks before every advance call:

```text
RequestAdvance()
  → IsTypewriterActive()?
      True  → SkipTypewriter()    ← first click: show full text
      False → AdvanceDialogue()   ← second click: next line / end dialogue
```

From the player's perspective: the first click on skip shows the full text immediately, the second click moves on.

## Building your own text widget

**Step 1** — Blueprint subclass: Parent `MayDialogueWidget_Text`, name e.g. `WBP_MyText`.

**Step 2** — UMG Designer: add a `RichTextBlock`, name: `DialogueRichText`. Register the decorator classes (see above).

> 📸 **Image placeholder:** `text-widget-umg-designer.png` — UMG Designer of WBP_MyText. Hierarchy: Canvas → RichTextBlock (Name: "DialogueRichText"). Details panel: font, colour, wrapping set, Decorator Classes populated.
> *Setup:* Open WBP_MyText in the UMG Designer, take a screenshot of the hierarchy and the Details panel section.

**Step 3** — Optional: override `OnCharacterRevealed` and `OnTypewriterComplete` in the Event Graph for custom effects.

**Step 4** — In your DialogFrame, replace the `TextWidget` child with `WBP_MyText`.

{% hint style="info" %}
Styling (font, colour, line spacing) is done through the `TextStyle` of the RichTextBlock in the UMG Designer. The typewriter effect itself is independent of this.
{% endhint %}
