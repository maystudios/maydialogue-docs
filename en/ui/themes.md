---
description: Horror, Visual Novel, and RPG themes — how to choose, activate, and customise a theme for your project.
---

# Themes & Starter Kits

{% hint style="warning" %}
**Starter themes (Horror / Visual Novel / RPG) ship as a separate content add-on and are not included in v1.0.** v1.0 ships with the bare `DialogFrame` widget base classes only. See the **Custom Themes** section below to build your own theme from scratch. Pre-built themes will be available in v1.1.
{% endhint %}

MayDialogue provides the widget framework. The **visual themes** you build as Blueprint subclasses — each is a complete set of widget classes that determines the appearance of your dialogue.

> 📸 **Image placeholder:** `themes-overview-comparison.png` — Three PIE viewport screenshots side by side: left Horror theme (dark, blood-red accents), centre VN theme (light panel at the bottom, large portrait on the left), right RPG theme (parchment-coloured banner, small portrait). Same dialogue text in all three.
> *Setup:* Start the same dialogue three times with different widget classes. Take screenshots in PIE.

## Which themes are available?

{% hint style="danger" %}
**Themes are a separate add-on and are not part of MayDialogue v1.0.** The following descriptions are a preview of the planned starter themes. In v1.0 only the base widget classes are available — use the "Building your own theme" section to get started right away.
{% endhint %}

| Theme | Style | Genre |
|---|---|---|
| **Horror** | Dark, blood-red accents, rough frame, nervous typewriter | Horror, Mystery |
| **Visual Novel** | Large portrait area, centred text box, soft animations | VN, story adventures |
| **RPG** | Classic dialogue box, name plate, inventory-compatible layout | RPG, Adventure |

The widget classes are subclassable — take a theme as a starting point and customise it.

---

## Activating a theme

Set this in Project Settings:

```ini
[MayDialogue]
DefaultDialogueWidgetClass = /Game/UI/Themes/Horror/WBP_MayDialogue_Horror.WBP_MayDialogue_Horror_C
```

Done. The theme appears the next time a dialogue starts.

> 📸 **Image placeholder:** `themes-project-settings.png` — Project Settings → MayDialogue. Field `DefaultDialogueWidgetClass` with a filled widget path, red arrow pointing to it.
> *Setup:* Project Settings → MayDialogue, screenshot this section only.

---

## Horror theme

> 📸 **Image placeholder:** `theme-horror-ingame.png` — PIE viewport, Horror theme active. Dark panel at the bottom with subtle noise particles in the background, thin blood-red border lines, speaker name in a weathered hand-lettering font, text with shake effect on one word. Fade-in animation nearly complete (flicker fade).
> *Setup:* Set Horror theme widget, start a dialogue with a `<shake>` tag and emotion tag "Dialogue.Emotion.Scared". Take a screenshot shortly after the fade-in.

**Dialog frame**
- Background: pitch-black with a subtle noise overlay.
- Border: thin blood-red lines, slightly asymmetric.
- Fade-in animation: fast flicker fade (200 ms).
- Fade-out animation: glitch effect (pixels shift briefly, then fade out).

**Speaker**
- Portrait frame: cracked-glass look.
- Name font: hand-lettering or slightly skewed.
- Emotion "Scared": portrait switches + shake animation.

**Text**
- Typewriter: slightly irregular (variation via `<speed>` tags in content).
- Font: serif, slightly weathered.
- Shake on emotional words recommended: `<shake>Dead.</shake>`.

**Choice buttons**
- Hover: red glow, brief twitch.
- Disabled: nearly invisible, tooltip appears with a delay.

> 📸 **Image placeholder:** `theme-horror-choice-hover.png` — PIE viewport, Horror theme, PlayerChoice active. Button 1 hovered: red glow around the button area. Button 2 normal. Button 3 disabled (barely visible). Mouse cursor visible.
> *Setup:* Horror theme, PlayerChoice node with 3 choices (1 unavailable), hover the mouse over button 1, take a screenshot.

---

## Visual Novel theme

> 📸 **Image placeholder:** `theme-vn-ingame.png` — PIE viewport, VN theme active. Light, semi-transparent panel at the bottom centre (70% width). Large portrait on the left (256×256), speaker name as a badge above the text. Gentle slide-up animation complete. Three centred choice lines.
> *Setup:* Set VN theme widget, start a dialogue with a portrait and PlayerChoice. Take a screenshot after the fade-in is complete.

**Dialog frame**
- Background: 70% opacity, parchment colour, 12 px corner rounding.
- Position: bottom centred, 70% screen width.
- Fade-in animation: slide-up + 300 ms ease-out.

**Speaker**
- Portrait: 256×256, positioned on the left.
- Emotion tags control portrait switching via DataTable (emotion → texture).
- Name badge: above the text area, not inside the Speaker widget itself.

**Text**
- Typewriter at medium speed.
- Font: sans-serif, highly readable.

**Choice buttons**
- Centred hover lines.
- Optional: key hint (Q, W, E, R) at the end of each button.

> 📸 **Image placeholder:** `theme-vn-choice-layout.png` — PIE viewport, VN theme, three centred choice buttons. Each button has a key hint on the right ("Q", "W", "E"). Button 2 hovered (highlight colour). No disabled buttons in this screenshot.
> *Setup:* VN theme, PlayerChoice with 3 choices, key hints integrated in WBP_ChoiceButton, take a screenshot.

---

## RPG theme

> 📸 **Image placeholder:** `theme-rpg-ingame.png` — PIE viewport, RPG theme active. Parchment-coloured banner at the bottom (80% width), decorative border, name plate separately above the frame, small portrait on the left (64×64), dialogue text in a serif font. Three choice buttons with icons on the left (shield icon, speech icon, run icon).
> *Setup:* Set RPG theme widget, start a dialogue with a small portrait and 3 choices with different icons.

**Dialog frame**
- Background: semi-opaque, brown banner with decorative border.
- Position: bottom, 80% screen width.

**Speaker**
- Name plate: separate, above the frame, not inside it.
- Portrait: small (64×64), on the left.

**Text**
- Typewriter fast.
- Font: serif or a decorative italic.

**Choice buttons**
- Icons next to the choice text (combat icon, speech icon, magic icon).

---

## Building your own theme

A theme consists of Blueprint subclasses of all widget classes plus a top-level widget that composites them.

**Step 1 — Create subclasses**

Create Blueprint subclasses for all building blocks:

| Blueprint | Parent Class |
|---|---|
| `WBP_Theme_DialogFrame` | `UMayDialogueWidget_DialogFrame` |
| `WBP_Theme_Speaker` | `UMayDialogueWidget_Speaker` |
| `WBP_Theme_Text` | `UMayDialogueWidget_Text` |
| `WBP_Theme_ChoiceList` | `UMayDialogueWidget_ChoiceList` |
| `WBP_Theme_ChoiceButton` | `UMayDialogueWidget_ChoiceButton` |
| `WBP_Theme_SkipButton` | `UMayDialogueWidget_SkipButton` |

**Step 2 — Create assets**

- Background textures (noise, parchment, banner).
- Fonts (serif, hand-lettering, sans-serif).
- SoundCues for UI SFX (hover, click, dialogue open).

**Step 3 — Composite the top-level widget**

Create `WBP_MayDialogue_Theme` (Parent: `UMayDialogueWidget`). Insert the sub-widgets with the correct names and bind them via BindWidget.

**Step 4 — Activate**

```ini
DefaultDialogueWidgetClass = /Game/UI/Themes/YourTheme/WBP_MayDialogue_Theme.WBP_MayDialogue_Theme_C
```

> 📸 **Image placeholder:** `theme-custom-before-after.png` — Before/after split: left shows the default Slate debug widget (minimalist). Right: finished custom theme with all visual elements active. Same dialogue text, same speaker.
> *Setup:* Start the same dialogue once with Slate, once with the finished custom theme. Screenshots side by side.

## Multiple themes in the same project

For different dialogue contexts (main story vs. tutorial vs. cutscene) you swap the widget before starting the dialogue:

{% hint style="info" %}
A dedicated API hook for per-dialogue widget swapping is planned. For now: set the widget class in Settings before starting the dialogue, or build your own widget manager that swaps the widget at dialogue start.
{% endhint %}

End of UI chapter. Next: [Audio System](../audio/README.md).
