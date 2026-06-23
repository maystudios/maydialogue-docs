---
description: All fields of UMayDialogueEditorSettings — node colors, debug highlights, editor behavior.
---

# Editor Settings (Reference)

Compact reference for `UMayDialogueEditorSettings`. Only relevant in editor builds.

- **Class**: `UMayDialogueEditorSettings` (`UDeveloperSettings`)
- **Config file**: `EditorPerProjectUserSettings.ini`
- **Section**: `/Script/MayDialogueEditor.MayDialogueEditorSettings`
- **UI path**: *Edit → Editor Preferences → Plugins → MayDialogue Editor*

{% hint style="info" %}
These settings are **per user**, not per project — they are not checked into the repository. If your team wants consistent node colors, commit the values to `Config/DefaultEditorPerProjectUserSettings.ini`.
{% endhint %}

> 📸 **Image placeholder:** `editor-settings-panel.png` — Screenshot of the Editor Preferences panel.
> *Setup:* Editor open, Edit → Editor Preferences → Plugins → MayDialogue Editor. Full screenshot with color picker fields for all node types visible. A red arrow points to the navigation path and a single color picker button for illustration.

---

## Node Colors

All values are `FLinearColor`, displayed as color pickers in the panel. The respective node type group gets the title bar in that color.

| Property | Applies to Node(s) | Default |
|---|---|---|
| `SayLineColor` | SayLine | Bright blue |
| `PlayerChoiceColor` | PlayerChoice | Orange |
| `BranchColor` | Branch | Green |
| `RandomLineColor` | RandomLine | Purple |
| `WaitColor` | Wait | Grey |
| `LinkColor` | Link | Teal |
| `SubGraphColor` | SubGraph | Dark teal |
| `CameraFocusColor` | CameraFocus, CameraShake | Magenta |
| `AnimationColor` | PlayAnimation | Yellow |
| `VariableColor` | SetVariable | Mustard |
| `LogicColor` | Branch (logic fallback), FireEvent | Light green |
| `EntryExitColor` | Entry, Exit | White |
| `ErrorColor` | Nodes with compile errors (error tint) | Red |

> 📸 **Image placeholder:** `node-colors-overview.png` — Dialogue graph with one node of each type side by side.
> *Setup:* Open a dialogue asset that contains at least: Entry (white), SayLine (blue), PlayerChoice (orange), Branch (green), FireEvent (light green), Wait (grey), Exit (white). All nodes arranged compactly, no connections needed. Shows how the colors look in the real editor.

{% hint style="info" %}
Next to each color picker in the Editor Settings panel is an arrow icon that resets the value to the CDO default.
{% endhint %}

---

## Debug Highlights

Visible only during a PIE session in the debugger.

| Property | Type | Default | Meaning |
|---|---|---|---|
| `ActiveDebugColor` | `FLinearColor` | Bright yellow | Highlight border around the node where the Instance is currently paused. |
| `HistoryDebugColor` | `FLinearColor` | Pale yellow | Border around all nodes visited during this PIE session. |

> 📸 **Image placeholder:** `debug-highlights-pie.png` — Dialogue graph during PIE with an active debug session.
> *Setup:* Dialogue asset open in the editor, PIE running, dialogue active. One node is marked with a bright yellow border (current position). Two or three previous nodes have pale yellow borders (history). Screenshot shows the graph with these highlights.

---

## Editor Behavior

| Property | Type | Default | Meaning |
|---|---|---|---|
| `bAutoCompileOnSave` | `bool` | `true` | Saving a dialogue asset automatically triggers a compile pass. |
| `bShowMinimapByDefault` | `bool` | `false` | Reserved — the minimap feature is not yet active. |

---

## Layout & Auto-Grid

Controls **Auto-Layout** (Toolbar → Auto-Layout) and the optional **Auto-Grid**. See also [Comfort Features](../editor/comfort-features.md).

| Property | Type | Default | Meaning |
|---|---|---|---|
| `AutoLayoutHorizontalSpacing` | `float` | `420` | Horizontal distance (px) between successive node columns. |
| `AutoLayoutVerticalSpacing` | `float` | `180` | Minimum centre-to-centre distance (px) between nodes in a column. |
| `AutoLayoutInterRowGap` | `float` | `80` | Extra padding (px) above/below each node on top of its height. |
| `AutoLayoutGridSize` | `float` | `16` | Grid (px) that positions are snapped to after auto-layout (when Auto-Grid is off). |
| `AutoLayoutMaxBarycenterSweeps` | `int32` | `24` | Max. number of barycenter sweeps for crossing minimization. |
| `bAutoLayoutInsertReroutes` | `bool` | `true` | Auto-Layout inserts aligned reroute knots along long edges — **only at genuine bends** (already-straight edges are left alone). Auto-reroutes are rebuilt each run; your own reroutes are kept. Also a toolbar toggle. |
| `bEnableAutoGrid` | `bool` | `false` | Enables a coarser alignment grid (larger than the engine default of 16 px). On: nodes snap to `AutoGridSize` on drag, paste, creation, and auto-layout, and the background grid is drawn at that size. |
| `AutoGridSize` | `int32` | `64` | Grid size (px) used when Auto-Grid is enabled (16–512; a multiple of 16 is recommended). |

> Auto-Layout uses the Brandes–Köpf "Fast & Simple" method: connected nodes are aligned vertically so edges run as straight as possible, and node height prevents overlaps. The result is deterministic.

---

## Consistent Team Settings

1. Configure the settings locally as desired.
2. Extract the block from your `EditorPerProjectUserSettings.ini`:

```ini
[/Script/MayDialogueEditor.MayDialogueEditorSettings]
SayLineColor=(R=0.2,G=0.55,B=0.9,A=1)
PlayerChoiceColor=(R=0.95,G=0.55,B=0.1,A=1)
BranchColor=(R=0.2,G=0.7,B=0.2,A=1)
bAutoCompileOnSave=True
```

3. Commit these lines to `Config/DefaultEditorPerProjectUserSettings.ini`.

Team members get the same colors on their next repo pull, but can override them locally.

---

## Programmatic Access (C++/Editor Only)

```cpp
#if WITH_EDITOR
const UMayDialogueEditorSettings* ES = GetDefault<UMayDialogueEditorSettings>();
FLinearColor ChoiceColor = ES->PlayerChoiceColor;
#endif
```

Blueprint access does not exist — the class is editor-only.

---

## Reset After Plugin Update

If default colors have changed after a plugin update — simply delete the settings block from `EditorPerProjectUserSettings.ini` and restart the editor. The defaults are rewritten on first access.

## See Also

- [Project Settings](project-settings.md) — all runtime settings.
- [Editor → Debugger](../editor/debugger.md) — where the debug colors are visible in the graph.
