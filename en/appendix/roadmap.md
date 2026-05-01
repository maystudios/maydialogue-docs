---
description: Planned improvements and the long-term direction of the plugin.
---

# Roadmap

This page shows what is being worked on and where the plugin is heading. All items are **planned** — no dates are set. Priorities may shift.

{% hint style="info" %}
If you want to prioritize an item: open an issue in your team's tracker and explain the concrete project need. The roadmap is not a promise — it reflects the current assessment.
{% endhint %}

---

## Short-Term: Closing Open Gaps

### Fixed in v1.0

The following items from the [Known Issues list](../troubleshooting/known-issues.md) were closed in v1.0:

- ~~**Widget Lifetime**~~: ✅ Fixed — Subsystem cleans up Slate and UMG auto-widgets on Deinitialize.
- ~~**Wait Node Abort Cleanup**~~: ✅ Fixed — `AsyncState_Wait::Cleanup` stops all timers; Abort calls `CleanupPendingAsyncNodes`.
- ~~**PlayAnimation Abort Cleanup**~~: ✅ Fixed — `AsyncState_PlayAnimation::Cleanup` cleanly overrides the montage end delegate.
- ~~**Input Mode Restore (Slate)**~~: ✅ Fixed for the Slate path — `CachedPreviousInputMode` cached in the widget.
- ~~**FailBehavior in ExecuteNode**~~: ✅ Fixed — Node base and `Instance::ContinueToNode` consistently evaluate `FailBehavior`.
- ~~**`bOverride2D` on SayLine / PlaySound**~~: ✅ Replaced by the tri-state `NodeAudioMode` on both node types.
- ~~**SayLine `VolumeMultiplier` / `PitchMultiplier`**~~: ✅ Both fields are now available on SayLine.
- ~~**SetVariable Tag type**~~: ✅ Tag-type variables are supported in SetVariable.
- ~~**SetVariable Participant scope**~~: ✅ `Scope = Participant` + `TargetParticipantTag` implemented.
- ~~**Wait Node condition mode**~~: ✅ `WaitCondition` + `ConditionCheckInterval` implemented and polled in AsyncState.
- ~~**QuickSave helper**~~: ✅ `MayDialogueSaveHelper` provides QuickSaveToSlot / QuickLoadFromSlot / DeleteSlot / DoesSlotExist.
- ~~**Dialogue events → GameplayCues**~~: ✅ `LifecycleCueBindings` in settings, Subsystem routes them.
- ~~**Choice tag binding externally**~~: ✅ `OnChoiceMade` includes `ChoiceIndex` and `ChoiceTags`.
- ~~**Bridge Read/Write API**~~: ✅ Complete API on `IMayDialogueBridge` and Blueprint wrappers on the Subsystem.

### Still Open

- **Input Mode Restore (UMG path)**: `UMayDialogueWidget` doesn't yet have `ApplyDialogueInputMode` / `RestoreGameInputMode`.
- **Live Requirement Pills in Preview**: Choice and Branch pills in the Preview Runner should color-code Requirement status in real time.
- **Cross-Asset Step-Into in the Debugger**: Debugger follows the flow into linked assets.
- **Subsystem-level `OnAnyVariableChanged`**: Per-instance binding works; global forwarding is still missing.

---

## Medium-Term: Quality and Convenience

### UMG Starter Themes (v1.1)

Three finished widget template sets — not included in v1.0, planned for v1.1 as a separate content add-on:

- **Horror**: Dark layout, red accents, pixelated font style.
- **Visual Novel**: Large portrait area, soft fade-in/out animation.
- **RPG**: Classic dialogue box with name plate and icon slot.

### Bridge Extensions

- `OnAnyVariableChanged` global forwarding on Subsystem (per-instance binding already works).
- `OnNodeReached` delegate on the Bridge (not just on the Instance).
- Read API for current scope info.
- Write API for save/restore snapshots of Instance variables — useful for checkpoint systems.

### Babel Polish

- Sample caching for procedural blips (performance improvement).
- Parameterizable phoneme prosody curves for more natural voice profiles.
- Per-speaker profiles as DataTable for fast project-wide variance.

### Editor Features

- **Live Validation**: Optional asynchronous debounce check after changes.
- **Minimap Tab**: Dedicated tab alongside the Outline panel.
- **Cross-Asset Navigation**: Editor automatically opens the target asset when you click a Link node.

### Replication

- `ClientUpdateConversation` RPC with a net-serializable message.
- Net-safe struct variants for dialogue messages and choice entries.
- Multiplayer paths (host + client) verified via integration tests.

> 📸 **Image placeholder:** `roadmap-theme-preview.png` — Three widget themes side by side in PIE.
> *Setup:* Three PIE windows open, each with the Horror, VN, and RPG theme. The same SayLine "What do you want here?" is displayed in all three themes. Visible are the differences in background color, font, portrait position, and button style.

---

## Long-Term: Extended Feature Set

### DataTable Integration for Speakers

Speaker definitions as DataTable rows, allowing project-wide changes in a single place:

```text
Speaker_Guard
├── DisplayName: "Guard"
├── Portrait: P_Guard
├── NodeColor: #8B0000
├── SoundClass: SC_VoiceGuard
├── Attenuation: Att_NPC
└── BabelProfile: BP_Babel_Guard
```

Assets reference the row instead of holding their own speaker struct. Changes to the speaker propagate automatically to all assets.

### Visual Preview Widget

A WYSIWYG preview panel directly in the asset editor: SayLines are rendered with portrait, emotion tag visualization, and typewriter — without needing to start the Preview Runner.

### Import / Export

- **Import from Articy, Yarn Spinner, Twine**: Port existing dialogue scripts to MayDialogue.
- **Export as JSON**: For external script analysis, localization pipelines, and quality assurance.

### Community Node Extensions

A structured format for community-contributed nodes — custom Requirement, SideEffect, and Action classes as distributable share packages.

---

## Non-Goals

The plugin stays focused on its core area. The following topics are intentionally excluded:

- No quest system (separate responsibility).
- No cutscene sequencer.
- No voice recording or voice casting tool.
- No multiplayer UX (shared sessions, voting).
- No live collaboration editor.

Features that go beyond the dialogue scope belong in separate tools — to keep the plugin lean and maintainable.
