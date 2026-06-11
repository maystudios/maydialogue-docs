---
description: Swap the dialogue UI theme at runtime — per-level with a drop-in actor, event-driven mid-conversation, or programmatically from Blueprint/C++.
---

# Runtime Theme Switching

## Scenario

Your game ships with several dialogue looks — a grimy Horror frame, a soft Visual-Novel pill, a gilded RPG panel — and you want a different one per level, or you want to switch between them *while the game is running*. MayDialogue makes both first-class:

* **`AMayDialogueThemeSetter`** — a drop-in level actor. Pick a theme widget class and every dialogue in that level uses it. No code, no Blueprint. Optionally map event tags to themes for **live, mid-conversation** switching.
* **`UMayDialogueSubsystem::SetDialogueWidgetClassOverride`** — a world-scoped widget-class override you can call from Blueprint or C++ for fully programmatic switching (settings menu, per-NPC look, scripted cutscene).

Both routes set the same world-scoped override. If a dialogue is already on screen, setting it **live-switches** the UI immediately — the old widget is torn down and the new one is spawned and rebound to the running conversation. The showcase map's **Theme Kiosk** demonstrates the event-driven path end to end: the **Curator** dialogue offers three choices, each firing a `FireEvent` side effect (`MayDialogue.Demo.Theme.Horror`, `MayDialogue.Demo.Theme.VN`, `MayDialogue.Demo.Theme.RPG`), and the level's ThemeSetter re-skins the UI on the spot.

## What you will learn

* Drop an `AMayDialogueThemeSetter` into a level for a **static per-level theme** (zero code).
* Add an `EventThemes` table to that actor so a `FireEvent` tag **live-switches** the theme mid-conversation.
* Call `SetDialogueWidgetClassOverride` from Blueprint/C++ for **programmatic** switching (menus, per-NPC, cutscenes).
* (Appendix) The fully manual pattern — manage your own `UMayDialogueWidget` with `BindToInstance` — when you need total control.

## Prerequisites

* [Connecting a Custom UMG Widget](custom-umg-widget.md) completed — you understand that a theme is a `UMayDialogueWidget` subclass whose `BindWidgetOptional` slots are wired to themed component widgets.
* One or more theme widget classes ready (your own subclasses, or the bundled `WBP_MayDlg_Theme_Horror` / `_VN` / `_RPG`).

{% hint style="info" %}
**Precedence.** The world override always wins. The subsystem's UI decision tree is: **world override > `bUseSlateDialogueWidget` > `DefaultDialogueWidgetClass` > built-in `UMayDialogueWidget` fallback**. So a ThemeSetter (or a `SetDialogueWidgetClassOverride` call) overrides *both* the Slate debug widget and the project-wide default. The override is **world-scoped** — it lives and dies with the level's subsystem, so travelling to a level without a ThemeSetter automatically falls back to the project default.
{% endhint %}

## Path A — `AMayDialogueThemeSetter` (recommended)

### A1. Static per-level theme (no code)

1. In the level, **Place Actor → MayDialogue Theme Setter** (`AMayDialogueThemeSetter`).
2. In its Details panel, set **Theme Widget Class** to the theme for this level (e.g. `WBP_MayDlg_Theme_Horror`).

That is the whole setup. On `BeginPlay` the actor calls `SetDialogueWidgetClassOverride(ThemeWidgetClass)`, so every dialogue in that level opens in the chosen theme — without touching Project Settings. Leave **Theme Widget Class** unset to make the actor a no-op (the project default UI is used). The four shipped sample maps each carry one of these (Horror map → `_Horror`, VN map → `_VN`, RPG tavern + showcase → `_RPG`).

> 📸 **Image placeholder:** `runtime-theme-switch-themesetter.png` — `AMayDialogueThemeSetter` selected in a level, Details panel showing `Theme Widget Class = WBP_MayDlg_Theme_Horror`.
> *Setup:* Open the Horror sample map. Select the MayDialogue Theme Setter actor in the World Outliner; screenshot the Details panel with the `Theme Widget Class` field filled in.

### A2. Live, event-driven switching (`EventThemes`)

The same actor can switch the theme *mid-conversation* in response to a `FireEvent`. Fill in the **Event Themes** map (`TMap<GameplayTag, SoftClass<MayDialogueWidget>>`): when *any* dialogue in the world fires one of these tags, the mapped theme is applied immediately.

This is exactly how the showcase **Theme Kiosk** works:

| Event tag (`FireEvent`) | Theme widget class |
|---|---|
| `MayDialogue.Demo.Theme.Horror` | `WBP_MayDlg_Theme_Horror` |
| `MayDialogue.Demo.Theme.VN` | `WBP_MayDlg_Theme_VN` |
| `MayDialogue.Demo.Theme.RPG` | `WBP_MayDlg_Theme_RPG` |

Then, inside the dialogue, give each PlayerChoice a **FireEvent** side effect (or a standalone **FireEvent** action node) carrying the matching tag:

| Choice | FireEvent `EventTag` |
|---|---|
| "Show me the Horror frame" | `MayDialogue.Demo.Theme.Horror` |
| "Show me the Visual-Novel look" | `MayDialogue.Demo.Theme.VN` |
| "Show me the RPG panel" | `MayDialogue.Demo.Theme.RPG` |

Register the tags in `DefaultGameplayTags.ini` (the showcase content already does):

```ini
+GameplayTagList=(Tag="MayDialogue.Demo.Theme.Horror")
+GameplayTagList=(Tag="MayDialogue.Demo.Theme.VN")
+GameplayTagList=(Tag="MayDialogue.Demo.Theme.RPG")
```

When the player picks a choice, `FireEvent` broadcasts the tag on the running instance; the ThemeSetter is subscribed to every dialogue's event delegate, finds the matching entry in `EventThemes`, and calls `SetDialogueWidgetClassOverride` — which live-switches the on-screen UI to the new theme and rebinds it to the running conversation. No project glue, no Blueprint.

> 📸 **Image placeholder:** `runtime-theme-switch-eventthemes.png` — `AMayDialogueThemeSetter` Details panel with the `Event Themes` map expanded, three entries mapping the `MayDialogue.Demo.Theme.*` tags to the three theme widget classes.
> *Setup:* Open the showcase map. Select the Theme Kiosk's MayDialogue Theme Setter; screenshot the Details panel with the `Event Themes` map showing all three entries.

> 📸 **Image placeholder:** `runtime-theme-switch-graph.png` — Curator dialogue graph: a PlayerChoice whose three choices each carry a FireEvent side effect.
> *Setup:* Asset `DA_Showcase_Curator` open, PlayerChoice node selected, Details panel showing three `UMayDialogueChoice` entries; the first expanded with a `FireEvent` side effect whose `EventTag = MayDialogue.Demo.Theme.Horror`.

## Path B — `SetDialogueWidgetClassOverride` (programmatic)

When the trigger is *not* a dialogue event — a settings-menu button, a per-NPC look chosen at interaction time, a scripted cutscene — call the subsystem directly. One call does everything: it records the world-scoped override and, if a dialogue is running, tears the old UI down and brings up the new theme rebound to the live conversation. Pass **None** to clear the override; the project default returns with the next dialogue start.

**Blueprint:**

```text
[Get MayDialogue Subsystem]
   │
   ▼
[Set Dialogue Widget Class Override]
   └─ Widget Class: WBP_MayDlg_Theme_RPG   (soft class)
```

**C++:**

```cpp
void UThemeController::ApplyTheme(TSoftClassPtr<UMayDialogueWidget> ThemeClass)
{
    if (UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(this))
    {
        // Live-switches the on-screen UI if a dialogue is running;
        // otherwise applies from the next StartDialogue. None clears it.
        Sub->SetDialogueWidgetClassOverride(ThemeClass);
    }
}
```

The companion `GetDialogueWidgetClassOverride()` returns the currently active override (or None).

> 📸 **Image placeholder:** `runtime-theme-switch-setoverride-bp.png` — Settings-menu Blueprint: a button's OnClicked → `Get MayDialogue Subsystem` → `Set Dialogue Widget Class Override` with a soft theme class selected.
> *Setup:* A simple options-menu widget Blueprint. `OnClicked (RpgThemeButton)` → `Get MayDialogue Subsystem` → `Set Dialogue Widget Class Override` (Widget Class = `WBP_MayDlg_Theme_RPG`).

### Variations / going further

* **Per-NPC themes:** subscribe to the subsystem's `OnAnyDialogueStarted`, inspect the new instance's participants, and call `SetDialogueWidgetClassOverride` with that NPC's theme so each character opens in its own look.
* **Player-menu theme picker:** drive `SetDialogueWidgetClassOverride` from a settings button — the live switch makes the change visible even if a dialogue is currently open.
* **First-run default:** the override is empty until you set it, so the project-wide `DefaultDialogueWidgetClass` (or the Slate debug widget) still governs the "no theme chosen yet" start. Set a sensible default in Project Settings and let a ThemeSetter or `SetDialogueWidgetClassOverride` take over from there.

## Appendix — fully manual pattern (full control)

If you need to own the widget lifetime yourself — e.g. you want to composite the dialogue UI into a larger HUD, or drive an animation you control across the swap — you can skip the override entirely and manage your own `UMayDialogueWidget`. Bind it to the live conversation with the public `BindToInstance(Instance)` call and add/remove it from the viewport yourself:

```cpp
void UThemeController::SwapToManualWidget(TSubclassOf<UMayDialogueWidget> ThemeClass)
{
    UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(this);
    UMayDialogueInstance* Active = Sub ? Sub->GetActiveDialogue() : nullptr;

    // Tear down the widget we manage ourselves (if any).
    if (CurrentThemeWidget)
    {
        CurrentThemeWidget->UnbindFromInstance();
        CurrentThemeWidget->RemoveFromParent();
        CurrentThemeWidget = nullptr;
    }

    CurrentThemeWidget = CreateWidget<UMayDialogueWidget>(GetWorld(), ThemeClass);
    if (CurrentThemeWidget)
    {
        CurrentThemeWidget->AddToViewport();
        if (Active)
        {
            CurrentThemeWidget->BindToInstance(Active);
        }
    }
}
```

{% hint style="warning" %}
If you go fully manual, do **not** also set a world override for the same dialogue — the subsystem's auto-spawn and your manually managed widget would both bind to the same instance and you would see two panels. Pick one path: let the override own the UI (Paths A/B), or own it yourself (this appendix) and leave the override clear.
{% endhint %}

## How it fits together

```text
ThemeSetter (per-level)          ──┐
ThemeSetter EventThemes (live)   ──┤
SetDialogueWidgetClassOverride   ──┴─▶  world-scoped override
   │                                        │
   │                              (dialogue running?)
   │                                 yes → tear down old UI →
   │                                       spawn new theme → BindToInstance
   │                                 no  → applied at next StartDialogue
   ▼
Precedence: world override > bUseSlateDialogueWidget
            > DefaultDialogueWidgetClass > built-in fallback
```

## Troubleshooting

**The ThemeSetter does nothing.**
Its `Theme Widget Class` is unset (and `Event Themes` is empty), so the actor is a deliberate no-op. Fill in at least one. Also confirm the actor is actually in the loaded level and its `BeginPlay` ran.

**An `EventThemes` switch never fires.**
The dialogue is not firing the mapped tag, or the tag is misspelled. Confirm the choice's `FireEvent` `EventTag` exactly matches a key in the ThemeSetter's `Event Themes` map and that the tag is registered in `DefaultGameplayTags.ini`.

**The theme only applies to the next conversation, not the current one.**
A live switch only happens while a dialogue is actually running. If you set the override before any dialogue starts (or after it ended), it simply applies at the next `StartDialogue` — that is expected.

**Two dialogue panels show at once.**
You combined the world override with the manual appendix pattern, so both an auto-spawned widget and your own widget are bound to the same instance. Use one path only (see the appendix warning).
