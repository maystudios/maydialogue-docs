---
description: What happens from start to exit — and how cleanup works.
---

# Instance & Lifecycle

The dialogue asset is the blueprint. The **running conversation** is an instance — a short-lived object that exists for exactly one dialogue. This chapter shows what phases an instance passes through and what you can do at each point.

## The lifecycle at a glance

```text
[No dialogue]
      │
      │  StartDialogue()
      ▼
   [Active]  ──── SayLine ready ──→  [Waiting for advance]
      │                                      │
      │  PlayerChoice ready                  │  AdvanceDialogue()
      ▼                                      │
[Waiting for choice] ◄─────────────────────┘
      │
      │  SelectChoice(Index)
      ▼
   [Active]  ──── Exit node reached ──→  [Completed / Failed]
      │
      │  AbortDialogue() from outside
      ▼
   [Aborted]
```

The plugin uses a robust lifecycle system that ensures an instance always cleans up properly — regardless of how it ends.

> 📸 **Image placeholder:** `lifecycle-state-diagram.png` — State diagram as a graphic (not Mermaid code).
> *Setup:* Neatly designed flowchart with six states as boxes: Inactive, Active, WaitingForAdvance, WaitingForChoice, Completed, Aborted. Arrows labeled with triggering actions (StartDialogue, ReceiveMessage, AdvanceDialogue, SelectChoice, Exit node, AbortDialogue). Completed and Aborted have a thick border (terminal states).

## Start phase: What happens during StartDialogue

When you call `StartDialogue(Asset, Instigator, Target)`, five steps run:

1. **Pre-flight check** — Does the asset have an Entry node? Is another dialogue already active?
2. **Create the instance** — A new instance is created and bound to the world.
3. **Resolve participants** — The speaker tags in the asset are matched against the Participant components in the level.
4. **Fire OnDialogueStarted** — Your code can now react.
5. **Navigate to the Entry node** — The first node is executed.

> 📸 **Image placeholder:** `blueprint-on-dialogue-started.png` — Blueprint graph reacting to OnDialogueStarted.
> *Setup:* BP graph of a GameMode or HUD actor. Event `OnDialogueStarted` (delegate bind) → `Set Input Mode UI Only` → `Show Mouse Cursor = true`. All pins labeled. Clearly: no manual widget creation needed — the plugin handles that itself.

## Node execution: What each node returns

After its execution, each node returns an instruction for how the dialogue should continue:

| Return value | What happens |
| --- | --- |
| Advance (Next Node) | Jump directly to the specified next node. |
| Pause + present choices | Instance waits. Widget shows choice buttons. |
| Return to last choice | Jumps back to the most recently presented PlayerChoice. |
| Return to dialogue start | Jumps back to the Entry. |
| Abort | Dialogue ends as Aborted. |

As a user you don't need to drive this mechanism directly — the plugin's nodes use it correctly internally.

## Requirements: When is a node executed?

Before a node is executed, the instance checks its requirements (if any).

| Result | What happens |
| --- | --- |
| Passed | Node is executed normally. |
| FailedButVisible | Node content is shown to the player but cannot be selected. |
| FailedAndHidden | Node does not appear at all. |

If a requirement fails, the node's `FailBehavior` field decides whether the dialogue skips or aborts.

## Choices: The interactive step

A `PlayerChoice` node builds its list in three steps:

1. **Build** — For each choice, requirements are checked and an entry with availability, text, and tags is created.
2. **Filter** — `FailedAndHidden` choices are removed.
3. **Present** — The instance waits. The widget shows the remaining choices.

When the player clicks → `SelectChoice(Index)`:

1. Availability is checked again (variables may have changed).
2. The choice's SideEffects are executed.
3. The dialogue jumps to the choice's target node.

> 📸 **Image placeholder:** `choices-ingame-widget.png` — In-game widget with active choice buttons.
> *Setup:* PIE mode, dialogue active. Widget shows three buttons at the bottom side by side. Button 1: "A friend of the king." (white, clickable). Button 2: "That's none of your business." (greyed out, tooltip: "Unavailable"). Button 3 is not visible (FailedAndHidden filtered out). Background: the level with the guard NPC.

## Async nodes: When a node waits

Some nodes pause the dialogue until an external event arrives:

- **Wait** — waits for a timer or an event tag.
- **PlayAnimation** — optionally waits until a montage ends.
- **CameraFocus** — waits for the end of the blend time.

The dialogue remains active in this state, but no new nodes are executed until the event fires.

{% hint style="warning" %}
If an async node gets stuck (event tag never fires, montage never ends), the instance will hang. The validator in the editor warns when an async node has no clear continuation.
{% endhint %}

## Links & sub-dialogues

A `Link` node or `SubGraph` node branches into another dialogue or an embedded sub-graph. The plugin remembers the return point.

When the linked dialogue ends:
- If there is a return point → the dialogue continues there.
- If there is none → the dialogue ends normally as Completed.

This allows dialogue fragments to be nested to any depth without you having to manage the return point yourself.

## End phase: Three ways to end

### Completed

The dialogue reaches an `Exit` node with status `Completed`:

1. All pending timers and event listeners are disconnected.
2. Running voice playback is stopped.
3. Camera blends back to the origin (if a CameraFocus was active).
4. `OnDialogueEnded` fires with status `Completed`.
5. The subsystem removes the instance at the end of the frame.

### Failed

Same as Completed, but the Exit node has status `Failed`. Useful for distinguishing "finished" from "failed" in the calling code.

### Aborted

Triggered from outside: a new dialogue starts, a level change occurs, or your code calls `AbortDialogue()`. Same cleanup as Completed, but status = `Aborted`.

> 📸 **Image placeholder:** `exit-node-details.png` — Details panel of an Exit node.
> *Setup:* Exit node selected in the graph. Details panel on the right shows: `ExitStatus = Completed` (dropdown). No further property entries. The node itself is visible as a red capsule in the graph.

## Delegate hooks: Where you can plug in

| Delegate | When | Typical use |
| --- | --- | --- |
| `OnDialogueStarted` | On start | Switch input mode, prepare camera |
| `OnMessageReceived` | Per SayLine | Start audio, set subtitles, trigger animation |
| `OnChoicesPresented` | When choices are ready | Render buttons (if using a custom widget) |
| `OnChoiceMade` | After selection | Analytics, achievements |
| `OnVariableChanged` | Per variable mutation | Notify quest system |
| `OnDialogueEnded` | On end | Restore input mode, trigger quest check |

All delegates are **multicast** — multiple systems can listen simultaneously.

> 📸 **Image placeholder:** `blueprint-on-dialogue-ended.png` — Blueprint graph for OnDialogueEnded.
> *Setup:* BP graph of a PlayerController. `Bind Event to OnDialogueEnded` → event fires, parameter `ExitStatus` checked with `Switch on EMayDialogueExitStatus` → Completed branch leads to `Quest Check Complete`, Aborted branch leads to `Restore Player Input`. All pins cleanly labeled.

## Summary

- An instance is created on start and cleaned up automatically at the end — no manual management needed.
- States: Active, WaitingForAdvance, WaitingForChoice, Completed, Aborted, Failed.
- Async nodes pause the dialogue; cleanup disconnects all listeners cleanly.
- `Link`/`SubGraph` enables nested dialogues without manual return code.
- The delegate hooks are the way external systems (quest, audio, analytics) react.

Next: [Participants & Speakers](participants-speakers.md).
