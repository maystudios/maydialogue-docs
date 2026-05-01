---
description: Current limitations and recommended approaches.
---

# Known Issues

As of: Plugin version **1.0.0**. This list is updated with each release.

{% hint style="info" %}
If you run into a problem not listed here, check the [Debugging Tips](debugging-tips.md) first. For new reports: include repro steps, a log excerpt, and the plugin version.
{% endhint %}

---

## Open Issues

### Widget & UI

| Problem | Recommended Workaround |
| --- | --- |
| UMG Starter Themes (Horror, VN, RPG) not yet included. | Set up your own theme (see [UI Architecture](../ui/umg-architecture.md)). |

### Audio

| Problem | Recommended Workaround |
| --- | --- |
| Babel synthesizer: audio quality and configurability (per-speaker depth, biquad filter) are being reworked. | Assign your own `BlipSounds` instead of using procedural mode. |
| Babel global off: the `bEnableBabelVoice` flag exists, but complete silence without a voice asset combined with the typewriter effect has not been fully verified. | Test in your own project and leave the `BlipSounds` array empty if needed. |

### Camera

| Problem | Recommended Workaround |
| --- | --- |
| CameraFocus FOV restore runs globally in Instance cleanup; the original FOV is not cached per node. | Use only a single FOV override per dialogue. |

### Editor

| Problem | Recommended Workaround |
| --- | --- |
| Cycle detection: the compiler reliably detects cycles, but the schema does not reject them in real time on connect. | Be careful manually when wiring Knot chains — the compiler reports an error on save. |
| Live validation is disabled by default (re-entry issue). | Press the Compile button after each change. |
| Minimap widget exists in code but is not assigned to its own editor tab. | Use the Outline panel as an alternative. |

---

## Resolved Issues

These items were previously open and are fixed in v1.0:

- Widget survives level teardown no longer: Subsystem Deinitialize correctly tears down both Slate and UMG auto-widgets.
- Wait timer on dialogue abort: `AsyncState_Wait::Cleanup` clears all timers.
- PlayAnimation montage end delegate bound after abort: `AsyncState_PlayAnimation::Cleanup` replaces the delegate.
- `EMayDialogueNodeFailBehavior` is now consistently evaluated in `ExecuteNode`.
- Wait node condition mode (polling) implemented: `WaitCondition` + `ConditionCheckInterval` in `MayDialogueNode_Wait.h`.
- SetVariable now writes both Dialogue and Participant scope via the `Scope` property.
- SetVariable supports the Tag type (`TagValue` branch) fully.
- Node-level 2D override: tri-state `NodeAudioMode` enum replaces the old `bOverride2D` flag on both SayLine and PlaySound.
- `VolumeMultiplier` / `PitchMultiplier` available on SayLine.
- Choice tag binding (`ChoiceTags` on the Choice sub-node) implemented; `OnChoiceMade` delegate passes ChoiceTags.
- QuickSave helper (`MayDialogueSaveHelper`) fully implemented.
- Input mode restore: both UI paths (Slate + UMG) reliably detect the previous input mode (`UIOnly` via `GameViewportClient::IgnoreInput()`, `GameAndUI` vs `GameOnly` via cursor state) and restore it exactly after dialogue ends.
- Rich-text decorators `<color>` and `<b>` functional.
- Component-based UMG widget variant (DialogFrame, Speaker, Text, ChoiceButton, ChoiceList, SkipButton) included.
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
