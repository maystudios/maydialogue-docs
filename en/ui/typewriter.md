---
description: The typewriter effect — how to configure speed, use control tags, and trigger skip-to-end.
---

# Typewriter Engine

The typewriter effect reveals dialogue text character by character. It supports inline tags for pauses and speed changes, and can be skipped to the full text at any time.

> 📸 **Image placeholder:** `typewriter-ingame-midreveal.png` — PIE viewport, dialogue active, typewriter running. Text approximately half revealed, visible cursor at the end of the revealed section. Visual rich text effect (e.g. a red word) already visible even though the characters after it have not been revealed yet.
> *Setup:* Start PIE, trigger a slow dialogue (CPS approx. 10), take a screenshot precisely while the typewriter is running.

## Basic configuration

In Project Settings → MayDialogue:

```ini
bEnableTypewriterEffect   = true   ; typewriter on/off. false = text appears immediately in full.
TypewriterCharsPerSecond  = 30.0   ; characters per second (global default)
```

If `bEnableTypewriterEffect = false`, the entire text appears immediately — useful for accessibility options.

> 📸 **Image placeholder:** `typewriter-project-settings.png` — Project Settings → MayDialogue. Fields `bEnableTypewriterEffect` (checkbox) and `TypewriterCharsPerSecond` (number 30.0) marked with red arrows.
> *Setup:* Edit → Project Settings → MayDialogue, screenshot this section only.

## Overriding speed per dialogue line

You can override the speed per dialogue line — either via the SayLine node in the editor or directly when calling:

```cpp
// In the text widget — CPS <= 0 uses the Settings default
TextWidget->StartTypewriter(Text, CharactersPerSecond);
```

## Control tags: pauses and speed changes in text

Two tags control the typewriter flow directly in the dialogue text. **They do not appear in the rendered text** — the parser consumes them before display.

| Tag | Effect |
|---|---|
| `<pause=X>` | Pauses the typewriter for X seconds |
| `<speed=X>` | Multiplies the speed from this point on. `<speed=1.0>` resets it |

### Examples

```text
He whispered: "She... she is dead."<pause=1.5> Silence.
```
→ After "dead." the typewriter pauses for 1.5 seconds before " Silence." appears.

```text
Fast: <speed=3.0>He ran. She ran. Everyone ran.<speed=1.0> And then: silence.
```
→ The middle section runs three times as fast, then normal speed resumes.

```text
You are dead. <pause=1.0><color=red><shake>Dead.</shake></color>
```
→ Pause, then red, shaking text.

> 📸 **Image placeholder:** `typewriter-pause-example.png` — Two PIE screenshots side by side: left shows the typewriter immediately after "dead." (pause active, no new character revealed), right shows " Silence." visible after the pause. Timestamp bottom-right.
> *Setup:* Trigger a dialogue with `<pause=1.5>`, take screenshots before and after the pause.

## Skip to end

When `SkipTypewriter()` is called (e.g. by clicking the skip button):

- The text jumps immediately to its complete form.
- All remaining `OnCharacterRevealed` events are **not** fired.
- `OnTypewriterComplete` fires once.

The top-level widget interprets the first advance click as a typewriter skip, and the second as a dialogue advance:

```text
RequestAdvance()
  → IsTypewriterActive() = true  → SkipTypewriter()
  → IsTypewriterActive() = false → AdvanceDialogue()
```

## Relationship to rich text tags

Control tags and visual tags can be mixed freely:

| Tag | Processed by | Visible in text? |
|---|---|---|
| `<pause=X>` | Typewriter parser | No |
| `<speed=X>` | Typewriter parser | No |
| `<shake>...</shake>` | RichText decorator | Yes |
| `<wave>...</wave>` | RichText decorator | Yes |
| `<color=...>...</color>` | RichText decorator | Yes |
| `<b>...</b>` | RichText decorator | Yes |

Visual tags are active from the very first revealed character — the shake effect starts as soon as the first character of the `<shake>` section appears.

## Debug tips

| Problem | Solution |
|---|---|
| Text runs too fast | Check `TypewriterCharsPerSecond` in Project Settings |
| Tag appears as text | Space inside the tag? → `<pause= 0.5>` is wrong, correct: `<pause=0.5>` |
| Pause has no effect | The character index counts in the parsed text (without control tags). This is correct — check the spelling |
| `<speed>` does not reset | Set `<speed=1.0>` explicitly at the end of the section |

See [Rich Text Tags](rich-text-tags.md) for all visual tag options. For Babel binding on `OnCharacterRevealed`: [Audio → Babel System](../audio/babel-system.md).
