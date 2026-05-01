---
description: Open limitations of the current beta version and recommended workarounds.
---

# Known Issues

As of: Plugin version **0.1.0 (Beta)**. This list is updated with each release.

{% hint style="info" %}
If you run into a problem not listed here, check the [Debugging Tips](debugging-tips.md) first. For new reports: include repro steps, a log excerpt, and the plugin version.
{% endhint %}

---

## Open Issues

### Widget & UI

| Problem | Recommended Workaround |
| --- | --- |
| Static widget survives level teardown; doesn't re-bind after a level change. | Call `Subsystem → StopAllDialogues()` on level change, or manually remove the widget from the viewport. |
| UMG Starter Themes (Horror, VN, RPG) not yet included. | Set up your own theme (see [UI → Themes](../ui/themes.md)). |

### Nodes & Runtime

| Problem | Recommended Workaround |
| --- | --- |
| Wait timer survives dialogue abort on certain cleanup paths. | Keep wait durations short; or fire a timeout event to clean up early. |
| PlayAnimation montage end delegates remain bound after abort. | Set `bWaitForMontageEnd = false` on risky paths, or wire the animation end manually. |
| `EMayDialogueNodeFailBehavior` is declared but not yet consistently evaluated in `ExecuteNode`. | Place Requirements redundantly on outputs when an abort is expected. |
| Wait node condition mode (polling) missing in the UI path; only Duration and Event are available. | Use external Blueprint logic for the condition check, then call `FireEvent`. |

### Variables

| Problem | Recommended Workaround |
| --- | --- |
| SetVariable node currently writes only Dialogue scope; no direct write path to Participant scope. | Use a Blueprint SideEffect that calls `Participant → SetPersistentVariable` directly. |
| UI path for Tag-type variables is missing. | Use a String value with the GameplayTag path as a fallback. |

### Audio

| Problem | Recommended Workaround |
| --- | --- |
| Node-level 2D override (`bOverride2D`) missing on SayLine and PlaySound. | Set `AudioModeOverride = Force2D` at the speaker level. |
| `VolumeMultiplier` / `PitchMultiplier` missing on SayLine; only available on PlaySound. | Set at the speaker level. |
| Babel partially functional: sound quality, configurability, per-speaker depth in progress. | Assign your own `BlipSounds` instead of using procedural mode. |
| Babel global off: flag present, but complete silence without a voice asset not yet fully verified. | Test in your own project to confirm silence is achieved. |

### Camera

| Problem | Recommended Workaround |
| --- | --- |
| CameraFocus FOV restore runs globally in Instance cleanup; the original FOV is not saved per node. | Use only a single FOV override per dialogue. |

### Input

| Problem | Recommended Workaround |
| --- | --- |
| `RestoreGameInputMode()` hard-sets to `GameOnly`; the previous mode is not cached. | Manually set your own input mode after `OnDialogueEnded`. |

### Editor

| Problem | Recommended Workaround |
| --- | --- |
| Cycle detection: the compiler detects cycles, but the schema doesn't reject them on connect yet. | Be careful manually when wiring Knot chains. |
| Live validation disabled due to re-entry issue. | Press the Compile button after each change. |
| Minimap widget exists but is not assigned to any tab. | Use the Outline panel instead. |

### API & Bridge

| Problem | Recommended Workaround |
| --- | --- |
| Subscription hook for external systems on choice tags is incomplete. | Subscribe to the `OnChoiceMade` delegate, then look up ChoiceTags manually from `GetPendingChoices`. |
| Bridge write methods (`SelectChoice`, `ForceAdvance`, `SetVariable`) are exposed but not fully tested. | Write your own integration tests before release if the bridge is used critically. |
| QuickSave helper: API contract defined, implementation being finalized. | Use your project's own SaveGame with `ArIsSaveGame`. |

---

## Resolved Issues (Selection)

These items were previously open and have since been fixed:

- ApplyEffect implemented as an Action Node.
- Debugger Step-Into / Step-Out implemented.
- Participant `DefaultDialogue`, `SetActiveDialogue()`, `GetActiveDialogue()`, `StartDefaultDialogue()` available.
- Participant `AttenuationOverride` (AdvancedDisplay) available.
- 3-level audio settings (Plugin → Speaker → Node).
- Audio playback in the editor preview with culture selection.
- Preview Runner and FindResults as separate tabs.
- Auto-layout button (Sugiyama algorithm).
- Knot cycle detection in the Slate widget.
- Compiler `ResolveKnotChain` VisitedSet.
- Compiler choice target GUID invalidation.
- Unlimited immediate transitions fixed (MaxHops + VisitedNodes).
- Random Line state is now per-Instance (no longer on the asset node).
- Choice timeout state cleared on new `PresentChoices`.
- Subsystem Deinitialize cleans up completed dialogues.
- Validator checks async nodes without continuation.
- Failed dialogue start: entry point check before Instance creation.
- Rich-text decorators `<color>` and `<b>` functional.
- Component-based UMG widget variant included.

---

## Reporting a New Issue

Use the GitHub Issues tracker or reach out in the support channel. Always include:

1. **Repro steps** — What to click, what to expect, what happens.
2. **Log excerpt** — Only the relevant lines from `Saved/Logs/`.
3. **Plugin version** — Visible in `MayDialogue.uplugin` under `VersionName`.
