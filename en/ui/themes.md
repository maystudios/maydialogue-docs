---
description: Horror, Visual Novel, and RPG themes — how to choose, activate, and customise a theme for your project.
---

# Themes & Starter Kits

{% hint style="success" %}
**Three production themes ship with the plugin** at `/MayDialogue/Themes` — Horror (**Pale Static**), Visual Novel (**Daybreak**), and RPG (**Gilded Slate**) — plus a neutral `WBP_DialogueTheme_Default` under `/MayDialogue/Samples/UI`. Every sample map applies its theme automatically through an `AMayDialogueThemeSetter` actor, so you can play any sample level and see a finished theme immediately. Take any of them as a starting point and subclass it, or build your own from scratch with the **Building your own theme** section below.
{% endhint %}

MayDialogue provides the widget framework and ships finished themes on top of it. The **visual themes** are Blueprint subclasses — each is a complete set of widget classes that determines the appearance of your dialogue.

> 📸 **Image placeholder:** `themes-overview-comparison.png` — Three PIE viewport screenshots side by side: left Horror theme (near-black panel, typewriter font in the nameplate), centre VN theme (deep-violet panel, name pill, bright sans-serif), right RPG theme (dark slate panel with gilt border, Cinzel nameplate, ivory text). Same dialogue text in all three.
> *Setup:* Start the same dialogue three times with different widget classes (or play each sample map once — the ThemeSetter applies the theme automatically). Take screenshots in PIE.

## Which themes are available?

Three themes ship ready to use under `/MayDialogue/Themes`. Each is a complete set of subclassed widget classes plus a top-level composite (`WBP_MayDlg_Theme_Horror` / `_VN` / `_RPG`):

| Theme | Composite widget | Style | Genre |
|---|---|---|---|
| **Horror** — *Pale Static* | `WBP_MayDlg_Theme_Horror` | Near-black panel, pale text, typewriter font (Special Elite) in the nameplate, CRT flicker on open | Horror, Mystery |
| **Visual Novel** — *Daybreak* | `WBP_MayDlg_Theme_VN` | Deep-violet wide panel, name pill (Quicksand), Nunito Sans body text, choices float mid-screen | VN, story adventures |
| **RPG** — *Gilded Slate* | `WBP_MayDlg_Theme_RPG` | Dark slate panel with gilt border, Cinzel nameplate, ivory text, lock icon on locked choices | RPG, Adventure |

A neutral `WBP_DialogueTheme_Default` (under `/MayDialogue/Samples/UI`) is the plain baseline used when no theme is set. The widget classes are subclassable — take any theme as a starting point and customise it.

---

## Activating a theme

For a project-wide default, set this in Project Settings:

```ini
[MayDialogue]
DefaultDialogueWidgetClass = /MayDialogue/Themes/Horror/Widgets/WBP_MayDlg_Theme_Horror.WBP_MayDlg_Theme_Horror_C
```

Done. The theme appears the next time a dialogue starts. To apply a theme per level instead — or to switch live mid-conversation — use the `AMayDialogueThemeSetter` actor (see **Multiple themes in the same project** below), which is how every sample map sets its theme.

> 📸 **Image placeholder:** `themes-project-settings.png` — Project Settings → MayDialogue. Field `DefaultDialogueWidgetClass` with a filled widget path, red arrow pointing to it.
> *Setup:* Project Settings → MayDialogue, screenshot this section only.

---

## Horror theme

> 📸 **Image placeholder:** `theme-horror-ingame.png` — PIE viewport, Horror theme active (play the `L_Horror_Corridor` sample map). Near-black panel (1120×240) bottom-centred, speaker nameplate above-left in the Special Elite typewriter font, pale grey-white text. Fade-in animation nearly complete (CRT flicker fade).
> *Setup:* Start `L_Horror_Corridor` in PIE, walk up to the NPC (proximity start). Take a screenshot shortly after the fade-in.

**Dialog frame**
- Panel: near-black (`#131316`), 1120×240 @1080p, bottom-centred.
- Open animation: CRT flicker fade (~220 ms — the opacity jumps several times before settling).
- Close animation: fast fade (~160 ms).

**Speaker**
- Nameplate above-left of the frame; name in **Special Elite** (typewriter look), 20 pt, tracked-out spacing.
- A `PortraitImage` slot (96×96) exists in the widget but is collapsed by default — attach a texture via subclass if you want portraits.

**Text**
- Body font: Roboto 18 pt in pale grey-white (`#D8D4CC`).
- Typewriter effects via rich-text tags in the line text: `<shake>Dead.</shake>`, `<speed>` variations — see [Rich-text tags](rich-text-tags.md).

**Choice buttons**
- 720×56, stacked above the frame; dedicated Normal/Hover/Pressed/Disabled textures.
- Hover: opacity-nudge animation (`Anim_Hover`, ~80 ms).
- Locked-but-visible choices (`FailedButVisible`): a **lock icon** appears and the text switches to the disabled colour (`#8A857E`).

> 📸 **Image placeholder:** `theme-horror-choice-hover.png` — PIE viewport, Horror theme, PlayerChoice active. Button 1 hovered (hover texture + brightened text). One locked button with lock icon and dimmed text. Mouse cursor visible.
> *Setup:* Horror theme, PlayerChoice node with 3 choices (one with an unmet requirement, FailResult = Failed But Visible), hover the mouse over button 1, take a screenshot.

---

## Visual Novel theme

> 📸 **Image placeholder:** `theme-vn-ingame.png` — PIE viewport, VN theme active (play the `L_VN_Scene` sample map). Deep-violet wide panel (1400×300) bottom-centred, speaker name in a rounded pill in Quicksand, bright Nunito Sans body text. Soft fade complete.
> *Setup:* Start `L_VN_Scene` in PIE and walk up to Jun/Riley — the per-line speaker switching is part of the sample. Take a screenshot after the fade-in is complete.

**Dialog frame**
- Panel: deep violet (`#322B4D`), 1400×300 @1080p — the widest of the three themes.
- Open animation: soft fade (~280 ms), close ~220 ms.

**Speaker**
- Name **pill** (rounded) instead of a rectangular nameplate; name in **Quicksand SemiBold**, 17 pt.
- Generous `PortraitImage` slot (384×480) for VN portraits — collapsed by default; populate it with textures via subclass.

**Text**
- Body font: **Nunito Sans** 20 pt in near-white (`#F7F4FA`) — the only theme with its own body face.
- Typewriter at medium speed.

**Choice buttons**
- **Float mid-screen** (centre anchor) instead of sitting above the frame — the classic VN decision layout, 840×64 per line.
- Locked choices show the lock icon + disabled colour (`#A9A1BE`).

> 📸 **Image placeholder:** `theme-vn-choice-layout.png` — PIE viewport, VN theme, three choice lines floating mid-screen (centre anchor, detached from the frame). Button 2 hovered (hover texture). The panel with the running question is visible at the bottom.
> *Setup:* VN theme, PlayerChoice with 3 choices, hover the mouse over button 2, screenshot the whole viewport (so the floating layout is visible).

---

## RPG theme

> 📸 **Image placeholder:** `theme-rpg-ingame.png` — PIE viewport, RPG theme active (play the `L_DialogueShowcase` sample map, e.g. Gatekeeper Marrow). Dark slate panel (1260×232) with a gilt border bottom-centred, Cinzel nameplate above-left, ivory text. Three choice buttons above the frame, one locked with a lock icon.
> *Setup:* Start `L_DialogueShowcase` in PIE and walk up to Marrow — his dialogue ships with a locked-but-visible gate choice out of the box.

**Dialog frame**
- Panel: dark slate (`#161A21`) with a gilded 9-slice border, 1260×232 @1080p, bottom-centred.
- Open animation: fade ~200 ms, close ~150 ms.

**Speaker**
- Nameplate above-left of the frame; name in **Cinzel** (engraved-capitals look), 17 pt, slightly tracked out.
- `PortraitImage` slot (96×96) present, collapsed by default.

**Text**
- Body font: Roboto 18 pt in ivory (`#EDE4D3`) — the text carries the parchment feel, not the panel.
- Typewriter fast.

**Choice buttons**
- 560×52, stacked above the frame, gilded hover texture.
- Locked-but-visible choices: **lock icon** + disabled colour (`#978F7F`) — experienced first-hand in the Marrow sample.

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

`DefaultDialogueWidgetClass` (above) is the project-wide default, but you are **not** limited to one theme per project. The subsystem holds a **world-scoped widget-class override** that lets you pick a theme per level, or switch live mid-conversation.

**Precedence (which UI the subsystem auto-spawns):**

```text
world override  >  bUseSlateDialogueWidget  >  DefaultDialogueWidgetClass  >  built-in UMayDialogueWidget fallback
```

You set the world override two ways:

* **`AMayDialogueThemeSetter`** — a drop-in level actor. Set its `Theme Widget Class` for a static per-level theme (zero code), and/or fill its `EventThemes` map so a dialogue's `FireEvent` tag live-switches the theme mid-conversation.
* **`UMayDialogueSubsystem::SetDialogueWidgetClassOverride(WidgetClass)`** — call from Blueprint/C++ for programmatic switching (settings menu, per-NPC look, cutscene). If a dialogue is on screen, the UI live-switches immediately (old widget torn down, new one spawned and rebound); passing **None** clears the override back to the project default at the next dialogue start.

The override is world-scoped, so travelling to a level without a ThemeSetter automatically returns the project default. Full walkthrough: [Runtime Theme Switching](../recipes/runtime-theme-switch.md).

End of UI chapter. Next: [Audio System](../audio/README.md).
