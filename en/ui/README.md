---
description: How the MayDialogue UI is structured, when to use Slate, when to use UMG, and how to swap components.
---

# UI System

MayDialogue ships with two complete UI implementations. You decide which one you need.

| | Slate Debug Widget | UMG Components |
|---|---|---|
| Configuration | None | Blueprint subclasses |
| Portraits | No | Yes |
| Animations | No | Yes |
| Theming | Fixed | Flexible |
| Blueprint-extensible | No | Yes |
| When to use | Prototyping, fallback | Production |

> 📸 **Image placeholder:** `ui-overview-comparison.png` — Two viewport screenshots side by side: left the Slate debug widget (minimalist, white text on dark), right a UMG widget with portrait, animated frame, and styled buttons.
> *Setup:* Start PIE, play the same dialogue once with Slate and once with the UMG widget. Take screenshots in the PIE viewport.

## When Should I Use Which?

**Slate** — you want to write and test dialogues before the UMG design is ready. Zero configuration needed, playable immediately.

**UMG** — you are building the final game. You want portraits, animations, custom fonts, custom button styles, and the freedom to swap any component for your own Blueprint class.

## How Is the UMG System Structured?

Five widget classes, each of which you can individually subclass and swap:

```
UMayDialogueWidget            ← Top-Level (orchestrates events)
└── UMayDialogueWidget_DialogFrame   ← Container + animations
    ├── UMayDialogueWidget_Speaker   ← Name + portrait + emotion
    ├── UMayDialogueWidget_Text      ← Typewriter text
    ├── UMayDialogueWidget_ChoiceList ← Answer buttons
    │   └── UMayDialogueWidget_ChoiceButton ← individual button
    └── UMayDialogueWidget_SkipButton ← Advance prompt
```

Each class is `Abstract, Blueprintable`. You create a Blueprint subclass, design the appearance in the UMG Designer, and the system wires everything automatically via `BindWidget`.

## How Do I Swap a Component?

For example, if you want to swap the Speaker widget for your own:

1. Create a new Blueprint widget — **Parent Class:** `MayDialogueWidget_Speaker`.
2. Build the layout in the UMG Designer (PortraitImage, NameText as you like).
3. Implement the Event `On Speaker Changed` for custom emotion logic.
4. Register the new class in your top-level widget as the `SpeakerWidget` slot.

The same applies to all other components. You can swap individual components without touching the rest.

## Chapter Overview

* [Slate Debug Widget](slate-debug-widget.md) — the ready-to-use fallback.
* [UMG Architecture](umg-architecture.md) — composition, configuration paths, common errors.
* [Dialog Frame](dialog-frame.md) — container, animations, interface.
* [Speaker Widget](speaker-widget.md) — name, portrait, emotion tags.
* [Text Widget](text-widget.md) — typewriter binding, rich text.
* [Choice List & Choice Button](choice-list.md) — answer buttons, layout options.
* [Skip Button](skip-button.md) — advance prompt, platform hints.
* [Typewriter Engine](typewriter.md) — speed, tags, skip-to-end.
* [Rich-Text Tags](rich-text-tags.md) — `<pause>`, `<speed>`, `<shake>`, `<wave>`, `<color>`, `<b>`.
* [Themes & Starter Kits](themes.md) — Horror, Visual Novel, RPG.
