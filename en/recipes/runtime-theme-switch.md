---
description: Swap the dialogue UMG theme at runtime — driven by a FireEvent side effect from inside the dialogue itself.
---

# Runtime Theme Switching

## Scenario

Your game ships with several dialogue looks — a grimy Horror frame, a soft Visual-Novel pill, a gilded RPG panel — and you want to switch between them while the game is running. The showcase map's "Theme Kiosk" demonstrates exactly this: the **Curator** dialogue offers three choices, each firing a `FireEvent` side effect (`MayDialogue.Demo.Theme.Horror`, `MayDialogue.Demo.Theme.VN`, `MayDialogue.Demo.Theme.RPG`), and a small piece of project glue reacts by re-skinning the dialogue UI.

This recipe documents the **working, shipped-API approach**. Read the honesty box below first: MayDialogue does **not** ship a single "swap the live widget" call, so the pattern is "pick the theme class, then (re)create the widget" — and that is what the kiosk really does.

{% hint style="warning" %}
**What the shipped API does and does not give you (verified against source, 1.0):**

* There is **one** widget-class setting — `UMayDialogueSettings::DefaultDialogueWidgetClass` (`TSoftClassPtr<UMayDialogueWidget>`). A "theme" is one `UMayDialogueWidget` subclass (e.g. `WBP_MayDlg_Theme_Horror`) whose `BindWidgetOptional` slots are pre-wired to themed component widgets. Pointing the plugin at a theme = setting this one class.
* The subsystem auto-spawns that widget **once**, on the first dialogue start, into `AutoSpawnedUMGWidget`. That field is **private with no public getter or setter**, and there is **no** `SetTheme()` / `ApplyTheme()` / `RebuildComponents()` method on `UMayDialogueWidget`. So you cannot restyle the *already-spawned* auto-widget in place through the public API.
* What you **can** do with shipped API: manage your own `UMayDialogueWidget`, bind it with the public `BindToInstance(Instance)` call, and add/remove it from the viewport. That is the clean runtime swap. The honest workaround for the *current* conversation is "recreate the themed widget and rebind it"; for the *next* conversation, writing `DefaultDialogueWidgetClass` changes what auto-spawn uses.
{% endhint %}

## What you will learn

* Author the three theme choices as `FireEvent` side effects on a PlayerChoice.
* Subscribe **once** to the subsystem's `OnAnyDialogueStarted` to capture every new `UMayDialogueInstance`.
* React to the per-instance `OnDialogueEventFired` (a `FGameplayTag`) and map the theme tags to widget classes.
* Drive your own themed `UMayDialogueWidget` so the swap is visible immediately, with no plugin-side widget-swap API.

## Prerequisites

* [Connecting a Custom UMG Widget](custom-umg-widget.md) completed — you understand that a theme is a `UMayDialogueWidget` subclass.
* The three bundled themes exist under the plugin's `Content/Themes/{Horror,VN,RPG}/Widgets/WBP_MayDlg_Theme_*` (or your own subclasses).
* `Project Settings → MayDialogue → UI → Use Slate Dialogue Widget` is **off** (`bUseSlateDialogueWidget = false`) so the UMG path is active.

## Step by step

### 1. Define the theme event tags

In `DefaultGameplayTags.ini` (the showcase content already registers these):

```ini
+GameplayTagList=(Tag="MayDialogue.Demo.Theme.Horror")
+GameplayTagList=(Tag="MayDialogue.Demo.Theme.VN")
+GameplayTagList=(Tag="MayDialogue.Demo.Theme.RPG")
```

### 2. Fire the theme event from the dialogue

Open the curator dialogue. On the PlayerChoice node, give each choice a **FireEvent** side effect (or wire a standalone **FireEvent** action node after the choice). The only field that matters is the tag:

| Choice | FireEvent `EventTag` |
|--------|----------------------|
| "Show me the Horror frame" | `MayDialogue.Demo.Theme.Horror` |
| "Show me the Visual-Novel look" | `MayDialogue.Demo.Theme.VN` |
| "Show me the RPG panel" | `MayDialogue.Demo.Theme.RPG` |

`FireEvent` broadcasts the tag on the running instance — it reaches every listener bound to that instance's `OnDialogueEventFired`. Nothing in the dialogue itself touches the UI; the project glue (next step) does the re-skin.

> 📸 **Image placeholder:** `runtime-theme-switch-graph.png` — Curator dialogue graph with a PlayerChoice whose three choices each carry a FireEvent side effect.
> *Setup:* Asset `DA_Showcase_Curator` open. PlayerChoice node selected, Details panel showing three `UMayDialogueChoice` entries; the first expanded with a `FireEvent` side effect whose `EventTag = MayDialogue.Demo.Theme.Horror`.

### 3. Map the tags to theme widget classes

In your theme controller (a GameInstance subsystem, the level Blueprint, or a small actor), build a lookup from event tag → `UMayDialogueWidget` subclass:

```text
MayDialogue.Demo.Theme.Horror → WBP_MayDlg_Theme_Horror
MayDialogue.Demo.Theme.VN     → WBP_MayDlg_Theme_VN
MayDialogue.Demo.Theme.RPG    → WBP_MayDlg_Theme_RPG
```

In Blueprint, a `Map<GameplayTag, SoftClass<MayDialogueWidget>>` variable is the simplest form. In C++, a `TMap<FGameplayTag, TSubclassOf<UMayDialogueWidget>>`.

### 4. Subscribe to every dialogue start

Bind once (e.g. on `BeginPlay`) to the subsystem's aggregate `OnAnyDialogueStarted`. It hands you the freshly created `UMayDialogueInstance`, which is where the event delegate lives.

**Blueprint:**

```text
[Event BeginPlay]
   │
   ▼
[Get May Dialogue Subsystem]
   │
   ▼
[Bind Event to On Any Dialogue Started]
        └─ Event:  HandleDialogueStarted(Instance)
```

**C++:**

```cpp
void UThemeController::Init()
{
    if (UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(GetWorld()))
    {
        Sub->OnAnyDialogueStarted.AddDynamic(this, &UThemeController::HandleDialogueStarted);
    }
}
```

### 5. On each start, bind the per-instance event

`OnAnyDialogueStarted` carries the `UMayDialogueInstance`. Subscribe to that instance's `OnDialogueEventFired` (one `FGameplayTag` parameter):

**Blueprint:**

```text
[HandleDialogueStarted (Instance)]
   │
   ▼
[Instance → Bind Event to On Dialogue Event Fired]
        └─ Event: HandleThemeEvent(EventTag)
```

**C++:**

```cpp
void UThemeController::HandleDialogueStarted(UMayDialogueInstance* Instance)
{
    if (Instance)
    {
        Instance->OnDialogueEventFired.AddDynamic(this, &UThemeController::HandleThemeEvent);
    }
}
```

### 6. Apply the theme — recreate and rebind your widget

When the theme tag arrives, look up the widget class and bring up that themed widget. Because there is no in-place restyle API, you remove the previous dialogue widget and create the themed one, then bind it to the live instance with the public `BindToInstance` call:

**C++:**

```cpp
void UThemeController::HandleThemeEvent(const FGameplayTag& EventTag)
{
    const TSubclassOf<UMayDialogueWidget>* Found = ThemeClasses.Find(EventTag);
    if (!Found || !*Found)
    {
        return; // not a theme tag — ignore
    }

    UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(GetWorld());
    UMayDialogueInstance* Active = Sub ? Sub->GetActiveDialogue() : nullptr;

    // Tear down the widget we manage ourselves (if any).
    if (CurrentThemeWidget)
    {
        CurrentThemeWidget->UnbindFromInstance();
        CurrentThemeWidget->RemoveFromParent();
        CurrentThemeWidget = nullptr;
    }

    // Bring up the themed widget and bind it to the running conversation.
    CurrentThemeWidget = CreateWidget<UMayDialogueWidget>(GetWorld(), *Found);
    if (CurrentThemeWidget)
    {
        CurrentThemeWidget->AddToViewport();
        if (Active)
        {
            CurrentThemeWidget->BindToInstance(Active);
        }
    }

    // Persist the choice so the NEXT dialogue auto-spawns in this theme too.
    if (UMayDialogueSettings* Settings = GetMutableDefault<UMayDialogueSettings>())
    {
        Settings->DefaultDialogueWidgetClass = *Found;
    }
}
```

The Blueprint equivalent: `Create Widget` (Class = the looked-up theme), `Add to Viewport`, then `Bind To Instance` (the active instance from `Get Active Dialogue`), and remove the previously created widget. Optionally write `DefaultDialogueWidgetClass` via a small C++ helper so the next conversation starts already themed.

> 📸 **Image placeholder:** `runtime-theme-switch-controller-bp.png` — Theme-controller Blueprint: OnAnyDialogueStarted → bind OnDialogueEventFired → switch on tag → Create Widget + Bind To Instance.
> *Setup:* GameInstanceSubsystem Blueprint graph. `OnAnyDialogueStarted` red event node → `Bind Event to On Dialogue Event Fired`. Below, a `Switch on GameplayTag` with three pins (Horror/VN/RPG), each feeding `Create Widget` → `Add to Viewport` → `Bind To Instance`.

### 7. Let the auto-spawn handle the "no theme chosen yet" start

For the very first dialogue, set `DefaultDialogueWidgetClass` to a sensible default (e.g. `WBP_MayDlg_Theme_VN`) in Project Settings. The subsystem auto-spawns it on the first dialogue start; your controller only takes over once a theme event fires. If you want your controller to own the widget from the start, leave the per-dialogue auto-spawn in place and simply replace it in step 6 — the auto-widget and your widget are separate `UMayDialogueWidget` instances, so unbind/remove yours and leave the auto one alone (or vice-versa); just avoid two widgets bound to the same instance at once.

## How it fits together

```text
Dialogue choice → FireEvent(MayDialogue.Demo.Theme.X)
        │ (broadcast on the running instance)
        ▼
Instance.OnDialogueEventFired(EventTag)
        │ (your controller listens, bound via OnAnyDialogueStarted)
        ▼
Look up theme class → CreateWidget → AddToViewport → BindToInstance
        │
        └─ (optional) write DefaultDialogueWidgetClass for the next conversation
```

## Variations / going further

* **Settings-only swap (between conversations):** if you do not need the *current* dialogue to re-skin mid-line, skip steps 4–6 and simply write `DefaultDialogueWidgetClass` when the theme tag fires. The new theme appears on the next `StartDialogue`. Simpler, but not live.
* **Player-menu theme picker:** drive the same `HandleThemeEvent` logic from a settings menu button instead of a dialogue choice — the swap path is identical once you have a tag (or a class) in hand.
* **Per-NPC themes:** map a speaker tag (not an event tag) to a theme and pick the widget class in `OnAnyDialogueStarted` based on the instance's participants, so each NPC opens in its own look.

## Troubleshooting

**Nothing happens when I pick a choice.**
The choice has no `FireEvent` side effect, or the tag is misspelled. Confirm the `EventTag` on the choice exactly matches `MayDialogue.Demo.Theme.*` and that the tag is registered in `DefaultGameplayTags.ini`.

**The event fires but the widget never changes.**
You are probably trying to restyle the plugin's auto-spawned widget — that is not exposed. Make sure step 6 *creates your own* `UMayDialogueWidget` and calls `BindToInstance`; the auto-widget is private and cannot be swapped in place.

**Two dialogue panels show at once.**
You added a new widget without removing the old one (auto-spawned or your previous theme widget). Call `RemoveFromParent` + `UnbindFromInstance` on the one you are replacing before adding the new one.

**The theme only applies to the next conversation, not the current one.**
That means you only wrote `DefaultDialogueWidgetClass` (the settings-only variation). To re-skin the live conversation, you must create and `BindToInstance` the new widget as in step 6.
