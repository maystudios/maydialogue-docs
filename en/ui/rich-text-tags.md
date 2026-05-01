---
description: The six core tags for rich text in dialogue — pause, speed, shake, wave, color, and b — with examples and visual effects.
---

# Rich Text Tags

MayDialogue supports six inline tags directly in dialogue text. Two of them control the typewriter flow; four produce visual effects.

## Overview

| Tag | Type | Effect |
|---|---|---|
| `<pause=X>` | Typewriter control | Pause for X seconds |
| `<speed=X>` | Typewriter control | Speed multiplier |
| `<shake>...</shake>` | Visual | Per-character jitter effect |
| `<wave>...</wave>` | Visual | Sinusoidal up-and-down movement |
| `<color=...>...</color>` | Visual | Colour override |
| `<b>...</b>` | Visual | Bold text |

Control tags (`pause`, `speed`) are consumed by the parser and do not appear in the rendered text. Visual tags remain in the text and are rendered by the `URichTextBlock`.

---

## `<pause=X>`

Pauses the typewriter for X seconds. Useful for dramatic pauses.

```text
He opened the door.<pause=1.5> Behind it: nothing.
```

> 📸 **Image placeholder:** `tag-pause-effect.png` — Two PIE viewport screenshots side by side. Left: text ends after "door." — typewriter paused, no new character revealed. Right: after 1.5 s " Behind it: nothing." appears in full. Timestamp in the corner.
> *Setup:* Start a dialogue with this example text, take screenshots immediately before and after the pause.

---

## `<speed=X>`

Multiplies the current typewriter speed from this point on. `<speed=1.0>` resets to the default.

```text
Normal.<speed=0.3> Veeeery slooowly...<speed=1.0> And back to normal.
```

> 📸 **Image placeholder:** `tag-speed-effect.png` — PIE viewport during the slow section. Text "Veeeery slooowly..." partially revealed, clearly showing the typewriter running very slowly. Comparison screenshot next to it at normal speed.
> *Setup:* Trigger a dialogue with `<speed=0.3>`, take a screenshot during the slow section.

---

## `<shake>...</shake>`

Per-character jitter effect. Each character moves randomly by a few pixels. The effect runs continuously.

```text
The hand<pause=0.5> <shake>trembled.</shake>
```

> 📸 **Image placeholder:** `tag-shake-effect.png` — PIE viewport, dialogue active. Text "trembled." is visible with per-character shake (characters slightly offset, irregular). Normal text next to it for comparison.
> *Setup:* Start a dialogue with `<shake>trembled.</shake>`, take a screenshot in PIE.

### Configuration in the Designer

`UMayDialogueShakeDecorator` has configurable properties:

```cpp
float ShakeIntensity  = 2.0f;  // Max. pixel offset
float ShakeFrequency  = 15.0f; // Jitter speed (times per second)
```

In the UMG Designer → `DialogueRichText` → Decorator Classes → select `UMayDialogueShakeDecorator` → Details panel.

---

## `<wave>...</wave>`

Sinusoidal up-and-down movement of characters. Feels smooth and flowing.

```text
<wave>The water rushed.</wave>
```

> 📸 **Image placeholder:** `tag-wave-effect.png` — PIE viewport, dialogue active. Text "The water rushed." with wave effect: characters at slightly different Y positions, sine shape visible. Screenshot must be taken during the animation.
> *Setup:* Start a dialogue with `<wave>The water rushed.</wave>`, take a screenshot in PIE (not paused).

### Configuration in the Designer

```cpp
float WaveAmplitude  = 3.0f;   // Max. vertical pixel offset
float WaveSpeed      = 3.0f;   // Cycles per second
float WaveCharOffset = 0.5f;   // Phase offset per character (in radians)
```

---

## `<color=...>...</color>`

Colours the enclosed text in a different colour.

```text
Hello <color=#FF4444>World</color>!
Warning: <color=red>Danger!</color>
```

### Supported formats

| Format | Example | Effect |
|---|---|---|
| Hex with `#` | `<color=#FF0000>` | Red |
| Hex without `#` | `<color=FF0000>` | Also red |
| Named colour | `<color=red>` | Basic colour names |

Supported named colours: `red`, `green`, `blue`, `yellow`, `white`, `black`, `cyan`, `magenta`, `orange`, `gray`.

> 📸 **Image placeholder:** `tag-color-effect.png` — PIE viewport, dialogue. Text "Hello " in white default colour, "World" in bright red, "!" white again. Below: "Danger!" in red using named colour. Both lines visible.
> *Setup:* Start a dialogue with this example text, PIE screenshot.

---

## `<b>...</b>`

Renders the text in bold. Purely typographic, no animation.

```text
<b>Important:</b> Never open the red door.
```

> 📸 **Image placeholder:** `tag-bold-effect.png` — PIE viewport, dialogue. "Important:" clearly bolder than "Never open the red door." next to it. Contrast between bold and normal clearly visible.
> *Setup:* Start a dialogue with this example text, PIE screenshot.

---

## Combining tags

Tags can be nested:

```text
<color=yellow><b>Warning!</b></color>
<shake><color=red>!!!</color></shake>
You are dead. <pause=1.0><color=red><shake>Dead.</shake></color>
```

Result of the last example: "You are dead." normally → pause 1 second → "Dead." in red with shake.

> 📸 **Image placeholder:** `tag-combined-effect.png` — PIE viewport, dialogue. Line "You are dead." normal white, then after the pause "Dead." in red with active shake effect. Screenshot must be taken after the pause.
> *Setup:* Start a dialogue with the combination example, take a screenshot after the pause.

---

## Registering decorators

For UMG widgets, the four visual decorators must be registered on the `URichTextBlock`:

1. Select `DialogueRichText` in the UMG Designer.
2. Details panel → **Decorator Classes** → `+`.
3. Add: `UMayDialogueShakeDecorator`, `UMayDialogueWaveDecorator`, `UMayDialogueColorDecorator`, `UMayDialogueBoldDecorator`.

For the Slate debug widget, the decorators are already registered automatically.

{% hint style="info" %}
**Custom decorators:** Create a Blueprint or C++ subclass of `URichTextBlockDecorator`. Implement `CreateDecorator` and add it to the decorator list of your RichTextBlock. Custom tags (e.g. `<flash>`, `<fade>`) are possible this way.
{% endhint %}
