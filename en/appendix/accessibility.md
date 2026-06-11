---
description: Larger text, colorblind-safe choice cues, screen-reader support, and remappable input — all built in.
---

# Accessibility

MayDialogue ships its dialogue UI with accessibility built in, so the players who need it are not locked out of your story. This page bundles every accessibility feature in one place: how to turn it on, where each setting lives, and exactly what it does under the hood.

Four areas are covered:

1. **Larger text** — a project-wide `DialogueTextScale` from 0.5× to 3.0×, applied on **both** UI layers.
2. **Colorblind-safe choice cues** — a non-color lock marker on unavailable choices, on by default.
3. **Screen-reader support** — accessible text on the speaker name, dialogue body, and every choice button, on both layers.
4. **Input remapping** — driven by Enhanced Input, so players can rebind the advance/skip/select keys.

Everything here works on the **built-in Slate debug widget** and on the **default UMG component widgets** with zero code. If you build a fully custom UMG widget from scratch, the same helpers are public so you can wire them in yourself — see the API reference below.

{% hint style="info" %}
These are the features behind SPEC §5.11. They are designed to meet the bar of a commercial plugin whose audience runs from solo indies to professional studios — accessibility is a shipping feature, not an afterthought.
{% endhint %}

---

## The no-code path

Open **Edit → Project Settings → Plugins → MayDialogue → Accessibility**. Two toggles cover the visual features; the rest is automatic.

1. Set **Dialogue Text Scale** to the size you want (e.g. `1.5` for 150% text). It applies the next time a dialogue line is shown.
2. Leave **Colorblind Safe Choice Cues** enabled (the default). Locked choices now show a 🔒 lock marker in front of the text.
3. Screen-reader text is emitted automatically by both widget layers — there is nothing to enable.
4. Input remapping comes from your Enhanced Input mapping context (see [Input remapping](#input-remapping-enhanced-input)).

That is the entire setup. No Blueprint, no C++, no widget replacement.

> 📸 **Image placeholder:** `accessibility-settings-panel.png` — The Accessibility category of the MayDialogue project settings.
> *Setup:* Editor open, Edit → Project Settings → Plugins → MayDialogue, scrolled to the **Accessibility** category. Both rows visible: `Dialogue Text Scale` (slider/field set to 1.5) and `Colorblind Safe Choice Cues` (checked). A red arrow points to the category header.

---

## Larger text — `DialogueTextScale`

`DialogueTextScale` is a single uniform multiplier applied to **every** dialogue font size: the message body, the speaker name, and the choice button labels. It is clamped to the range **0.5–3.0** (values outside are silently brought into range).

The value is read and clamped through the shared helper `MayDialogue::Accessibility::ClampTextScale(...)`, so the **Slate debug widget** and the **default UMG component widgets** apply exactly the same bounds — no widget drifts to a different scale. You do not need to replace any widget to use larger text: set the scale once and it applies project-wide.

{% hint style="info" %}
`DialogueTextScale` is honored on **both** layers: the Slate debug widget and the default UMG component widgets (Text, Speaker, ChoiceButton) all read it through `ClampTextScale`.
{% endhint %}

```ini
# DefaultGame.ini — 150% dialogue text everywhere
[/Script/MayDialogue.MayDialogueSettings]
DialogueTextScale=1.5
```

**Custom widgets:** if you wrote your own UMG widget that does not subclass the default component widgets, read `DialogueTextScale` from settings and pass it through `ClampTextScale` before applying it to your text blocks — that keeps you inside the supported range and consistent with the built-in widgets.

> 📸 **Image placeholder:** `accessibility-text-scale-compare.png` — Same dialogue line at 1.0× and at 2.0× side by side.
> *Setup:* Two PIE screenshots side by side of the same SayLine ("Halt! Who are you?") with two choices. Left: `DialogueTextScale=1.0`. Right: `DialogueTextScale=2.0`. The right panel shows visibly larger message text, speaker name, and choice buttons. Caption: "One setting scales the entire dialogue UI."

---

## Colorblind-safe choice cues — `bColorblindSafeChoiceCues`

When a choice is unavailable (its Requirement returned `FailedButVisible`), the UI normally dims it. Dimming alone relies on color/contrast perception. With `bColorblindSafeChoiceCues` enabled (the **default**), every unavailable-but-visible choice gets a **lock glyph prefix** (🔒, `U+1F512`) in front of its text, so the locked state is readable without perceiving color at all.

The prefix is produced by `MayDialogue::Accessibility::GetColorblindChoicePrefix(bEnabled, bIsAvailable)` and applied **before** the choice text, so it is visible in every theme regardless of button color. Both widget layers use the same helper, so available/locked choices look identical across Slate and UMG.

Disable it only if your project already has its own non-color availability indicator (a separate lock icon widget, a strike-through, etc.):

```ini
# DefaultGame.ini — keep the colorblind-safe lock marker on (default)
[/Script/MayDialogue.MayDialogueSettings]
bColorblindSafeChoiceCues=True
```

> 📸 **Image placeholder:** `accessibility-choice-lock-glyph.png` — A choice list with one available and one locked choice.
> *Setup:* PIE screenshot of a PlayerChoice with two entries. One choice is available ("Pay the toll"), one is locked because a Requirement fails ("Persuade the guard"). The locked choice shows the 🔒 prefix in front of its text. Caption: "Locked choices stay distinguishable without relying on color."

---

## Screen-reader support

Both widget layers emit **accessible text** for the three things a screen-reader user needs to follow a dialogue: who is speaking, what they said, and what choices exist. The text is built by three shared helpers so the wording is identical on Slate and UMG:

| Element | Helper | Announced as |
|---|---|---|
| Speaker name | `MakeSpeakerAccessibleText(SpeakerName)` | `Speaker: <name>` |
| Dialogue body | `MakeDialogueTextAccessibleText(MessageText)` | the plain (rich-tag-stripped) sentence |
| Available choice | `MakeChoiceAccessibleText(...)` | `Choice 1: <text>` |
| Locked choice | `MakeChoiceAccessibleText(...)` | `Locked choice 1: <text>` (plus `— <reason>` when a reason is set) |

The key detail for assistive technology: the **locked** announcement contains the words "Locked choice" and, when the Requirement supplies one, the requirement description — so a screen-reader user hears *why* a choice is unavailable even though they cannot see the 🔒 glyph or the dimmed color. The dialogue body is always the **plain** text (inline rich-text tags are stripped), so the reader speaks the sentence, not the markup.

These accessible strings are attached to the underlying Slate widgets when the engine is built `WITH_ACCESSIBILITY`. On builds without accessibility compiled in, the helpers still exist and return the same `FText`, but nothing is announced.

{% hint style="info" %}
This support is **designed for** the Windows screen readers (Narrator and NVDA) that read standard Slate accessible text. Engine accessibility coverage varies by platform and engine version; treat any platform outside that as untested until you have verified it in your own build.
{% endhint %}

> 📸 **Image placeholder:** `accessibility-screenreader-narrator.png` — Narrator reading a locked choice.
> *Setup:* PIE running with Windows Narrator enabled, focus on a locked choice button. The Narrator caption/overlay shows the spoken text "Locked choice 1: Persuade the guard — Requires Persuasion 3". Caption: "Locked choices announce the reason, not just the state."

---

## Input remapping (Enhanced Input)

Dialogue control — advance the line, skip the typewriter/voice, select a choice — is driven through **Enhanced Input**. That means the bindings live in your project's Input Mapping Context, so players can rebind them through whatever key-rebinding UI your game already exposes. MayDialogue does not hardcode an input device or a fixed key.

Full setup, the Input Actions involved, and how the plugin consumes them is covered in the runtime guide:

[→ Enhanced Input integration](../runtime/enhanced-input.md)

---

## API reference

All helpers live in the header `MayDialogueAccessibility.h` under the `MayDialogue::Accessibility` namespace. They are pure `FText`/`FString`/`float` helpers with no UObject or UMG dependency, so a custom Slate **or** UMG widget can include the header and reuse them.

| Symbol | Signature | Purpose |
|---|---|---|
| `ClampTextScale` | `float ClampTextScale(float InScale)` | Clamp `DialogueTextScale` to `[0.5, 3.0]`. Centralizes the bounds so both layers agree. |
| `GetColorblindChoicePrefix` | `FString GetColorblindChoicePrefix(bool bEnabled, bool bIsAvailable)` | Returns the 🔒 lock-glyph prefix when the setting is on **and** the choice is unavailable; empty string otherwise. |
| `MakeSpeakerAccessibleText` | `FText MakeSpeakerAccessibleText(const FText& SpeakerName)` | Builds `Speaker: <name>`. Returns empty `FText` for an empty name. |
| `MakeDialogueTextAccessibleText` | `FText MakeDialogueTextAccessibleText(const FText& MessageText)` | Returns the plain dialogue sentence for the reader. |
| `MakeChoiceAccessibleText` | `FText MakeChoiceAccessibleText(int32 ChoiceIndex, const FText& ChoiceText, bool bIsAvailable, const FText& UnavailableReason)` | Builds the per-choice announcement. `ChoiceIndex` is 0-based but printed 1-based. Prepends "Locked choice" and appends the reason when unavailable. |
| `LockGlyph` | `constexpr TCHAR LockGlyph[]` | The Unicode lock glyph (`U+1F512`) plus a trailing space, used as the colorblind prefix. |

### Settings these helpers read

| Setting | Type | Default | Where |
|---|---|---|---|
| `DialogueTextScale` | `float` | `1.0` (clamped `0.5–3.0`) | Project Settings → Plugins → MayDialogue → Accessibility |
| `bColorblindSafeChoiceCues` | `bool` | `true` | Project Settings → Plugins → MayDialogue → Accessibility |

Both fields are part of `UMayDialogueSettings` — see the full [Project Settings reference](../reference/project-settings.md).

---

## Edge cases & gotchas

- **The scale is read at display time, not at dialogue start.** Changing `DialogueTextScale` mid-PIE applies from the next line shown; it does not retro-resize a line already on screen.
- **Out-of-range scales are clamped silently.** `0.2` becomes `0.5`, `5.0` becomes `3.0`. There is no warning — `ClampTextScale` just brings the value into range.
- **`bColorblindSafeChoiceCues` only marks *visible* locked choices.** A choice whose Requirement returns `FailedAndHidden` is not rendered at all, so it gets no glyph — there is nothing to mark.
- **The lock glyph is plain text in the label.** If you measure choice text width yourself for layout, account for the extra glyph + space. The default widgets already do.
- **Custom widgets must opt in.** If you wrote a UMG widget that does **not** subclass the default component widgets, none of this is automatic — call the helpers yourself: `ClampTextScale` for your font sizes, `GetColorblindChoicePrefix` when building choice labels, and push the `Make*AccessibleText` strings onto your widgets via `SetAccessibleBehavior(EAccessibleBehavior::Custom, ...)` (the same call both built-in layers use under `WITH_ACCESSIBILITY`).
- **Screen-reader output needs `WITH_ACCESSIBILITY`.** Shipping builds that compile accessibility out will not announce anything even though the helper text is still produced.

---

## See Also

- [Project Settings (Reference)](../reference/project-settings.md) — every `UMayDialogueSettings` field, including both accessibility toggles.
- [Enhanced Input integration](../runtime/enhanced-input.md) — remappable dialogue controls.
- [UI → Choice List & Choice Button](../ui/choice-list.md) — where the colorblind prefix and accessible choice text are applied.
- [UI → Slate Debug Widget](../ui/slate-debug-widget.md) — the always-on default widget that honors all of the above.
