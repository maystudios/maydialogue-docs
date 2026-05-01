---
description: What the Slate debug widget shows, when to enable or disable it, and how to replace it with your UMG widget.
---

# Slate Debug Widget

`SMayDialogueWidget` is the built-in Slate UI. It starts automatically when no UMG widget is configured — with no additional setup.

> 📸 **Image placeholder:** `slate-widget-ingame.png` — PIE viewport: the Slate widget in a running dialogue. Visible: dark panel at the bottom, Speaker name highlighted in color (Speaker color from the graph), dialogue text with running typewriter, two Choice buttons numbered, blinking "Continue" hint at bottom right.
> *Setup:* Project Settings → MayDialogue → `bUseSlateDialogueWidget = true`, `DefaultDialogueWidgetClass = None`. Start PIE, trigger dialogue.

## What Does the Widget Show?

The layout from bottom to top:

```
[Viewport]
└── [Panel: dark, bottom-anchored, Slide-In animation]
    ├── [Portrait image] | [Speaker name with colored accent bar]
    │                   | [Divider line]
    │                   | [Dialogue text — Typewriter]
    │                   | [Choice buttons with numbering]
    └──────────────────── [Continue indicator: blinking]
```

No background art, no custom fonts, no animation extras — intentionally functional.

## When to Enable?

The widget is **active by default** when `DefaultDialogueWidgetClass` is left empty.

Typical scenarios:

- **Prototyping** — you write dialogues and test content before the UMG design is ready.
- **Debug builds** — UI polish is not a priority, but dialogues must be playable.
- **Automatic fallback** — your UMG widget is not set or has an error.

## Enabling / Disabling

In Project Settings → MayDialogue:

```ini
bUseSlateDialogueWidget = true    ; widget active
DefaultDialogueWidgetClass = None ; no UMG assigned
```

> 📸 **Image placeholder:** `slate-project-settings.png` — Project Settings → MayDialogue section. Fields `bUseSlateDialogueWidget` (checkbox, enabled) and `DefaultDialogueWidgetClass` (empty) are marked with red arrows.
> *Setup:* Edit → Project Settings → Plugins → MayDialogue. Screenshot only this section.

## Rich-Text Support

The Slate widget fully supports all four visual tags:

| Tag | Supported |
|---|---|
| `<shake>` | Yes |
| `<wave>` | Yes |
| `<color>` | Yes |
| `<b>` | Yes |
| `<pause>` | Yes (consumed by the Typewriter parser) |
| `<speed>` | Yes (consumed by the Typewriter parser) |

## Switching to UMG

When your UMG widget is ready:

1. Build your Blueprint widget based on `UMayDialogueWidget` (see [UMG Architecture](umg-architecture.md)).
2. Set `DefaultDialogueWidgetClass` in the Project Settings to your widget.
3. `bUseSlateDialogueWidget = false` (optional — the UMG widget automatically takes precedence).

The Slate widget is now out of the workflow.

{% hint style="warning" %}
**Level travel:** The Slate widget does not automatically rebind after a level change. Call `Subsystem->StopAllDialogues()` before travel, or remove the widget from the viewport manually. In Blueprint, the `Stop All Dialogues` Node on the MayDialogue subsystem is available for this.
{% endhint %}
