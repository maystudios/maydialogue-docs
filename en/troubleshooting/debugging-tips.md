---
description: Tools and methods for systematically debugging MayDialogue issues.
---

# Debugging Tips

If the symptom list in the common issues section doesn't help, proceed systematically. This page shows you the available tools and how to use them effectively.

---

## Output Log: Enable Verbose Logging

The fastest tool for tracking down problems is the Output Log with extended output.

**In the running editor / PIE via console:**

Press `~` (tilde) to open the UE console and enter:

```ini
log LogMayDialogue Verbose
```

From this point on, MayDialogue outputs detailed messages — node transitions, participant resolutions, requirement results.

**Permanently via DefaultEngine.ini:**

```ini
[Core.Log]
LogMayDialogue=Verbose
```

**Open the Output Log:** `Window → Developer Tools → Output Log`. Filter the log by `LogMayDialogue` to see only plugin messages.

> 📸 **Image placeholder:** `debug-output-log-filter.png` — UE Output Log with an active filter on LogMayDialogue.
> *Setup:* PIE active, dialogue triggered. Output Log window visible. The search field contains `LogMayDialogue`. Visible log lines show the dialogue flow: `[MayDialogue] StartDialogue: Asset DA_GuardConversation — Instance created`, `[MayDialogue] Node executed: SayLine_0 (Speaker: Guard)`, `[MayDialogue] Presenting choices: 2 visible, 0 hidden`.

### Important Log Messages and Their Meaning

| Message | Meaning |
| --- | --- |
| `StartDialogue: Asset X has no entry point` | Compile error or Entry node missing |
| `Cannot resolve participant with tag X` | Speaker tag doesn't match any actor |
| `Async node X timed out` | Wait node / PlayAnimation waiting too long |
| `Variable X not declared in asset` | SetVariable on an undeclared name |
| `Choice index out of bounds` | Programmatic `SelectChoice` call with invalid index |
| `Instance cleanup: aborted` | Dialogue was externally aborted |

---

## Dialogue Debugger: Breakpoints and Step Mode

The dialogue debugger is the most powerful way to isolate flow problems. It works during PIE.

**Set a breakpoint:**

1. Open the dialogue asset.
2. Right-click on the suspect node → **Toggle Breakpoint**.
3. Start PIE, trigger the dialogue.
4. On hit: the node lights up and the Debugger tab becomes active.
5. **Step Over** — executes the next node.
6. **Step Into** — enters sub-graphs or linked assets (cross-asset, if available).
7. **Continue** — runs until the next breakpoint.

> 📸 **Image placeholder:** `debug-debugger-active.png` — Dialogue Debugger tab during an active breakpoint.
> *Setup:* Dialogue asset open, PIE active. Debugger tab visible. A SayLine node is highlighted yellow (active state). In the Debugger tab on the right: variable watch list with `bMetGuard = false`, `ChosenGreeting = ""`. Buttons Step Over, Step Into, Continue are visible and active.

**Using the Variable Watch:**

In the Watch panel you can see variables live and override them manually. Useful for checking:

- Is a Requirement failing because a Bool value is `false` when it should be `true`?
- Which branch path is being taken?

Click on a variable value in the Watch panel and enter a new value — the next node step will use it.

---

## Preview Runner: Iteration Without PIE

The Preview Runner plays the dialogue directly in the asset editor — no PIE start, no Participant components needed.

**When it's most useful:**

- Rapid testing of text, flow order, and branch logic.
- Simulating tag states without a GAS setup.
- Checking whether a problem lies in the asset itself or in the game setup.

**Diagnostic rule:**

```
Does the dialogue run correctly in the Preview Runner?
  │
  ├─ Yes → Problem is in the PIE setup (participants, tags, widget, input)
  └─ No  → Problem is in the asset itself (nodes, links, requirements)
```

> 📸 **Image placeholder:** `debug-preview-runner.png` — Preview Runner panel with an active dialogue.
> *Setup:* Dialogue asset open. Preview Runner tab active. Visible: SayLine text "Halt! Who goes there?", speaker name "Guard", two choice buttons below. On the left in the panel: simulated tag list with `Story.Found.Codex = true`. The first choice button is green (requirement met), the second is grey (FailedButVisible).

**Setting simulated tags in the Preview Runner:**

The Preview Runner panel has a Tags area. Add GameplayTags there to simulate Requirements without going into PIE. This lets you test all branches without needing real GAS setups.

---

## Outline Panel: Quickly Locate Errors

If the compiler reports an error but you don't know where in the graph it is:

1. Open the **Outline panel** (tab in the asset editor).
2. Filter by node type (e.g. show only SayLines).
3. Click on entries — the graph jumps to the node's position.

With **Ctrl+F** (Find in Dialogue) you can search for text, speaker tags, or comments. Useful in large graphs with many nodes.

> 📸 **Image placeholder:** `debug-outline-panel.png` — Outline panel with a filter and a list of all SayLine nodes.
> *Setup:* Large dialogue asset open (20+ nodes). Outline tab active. Filter dropdown set to "SayLine". List shows all SayLines with speaker tag and a preview of the first sentence. One entry is highlighted red (validator error: missing voice asset).

---

## Fall Back to the Slate Widget

If you're unsure whether the problem is in the UMG widget or deeper:

1. **Project Settings → MayDialogue → UI** → `bUseSlateDialogueWidget = true`, `DefaultDialogueWidgetClass = None`.
2. Restart the editor, start PIE, trigger the dialogue.

| Result | Meaning |
| --- | --- |
| Dialogue visible in the Slate widget | Bug is in the UMG widget setup |
| Dialogue also invisible in the Slate widget | Bug is deeper (runtime, participant, subsystem) |

---

## Isolation Test: Bisect

If a large asset is broken and you don't know where:

1. Create a **copy** of the asset (right-click → Duplicate).
2. Delete the back half of the nodes in the copy.
3. Compile and test.
4. Error gone? The problem was in the deleted half.
5. Error still there? The problem is in the remaining half.
6. Repeat with the remaining half — **binary search at the node level**.

---

## Crash Analysis

For a real UE crash:

1. Open the crash log: `Saved/Logs/` in the project folder, newest `.log` file.
2. Search the stack trace for `UMayDialogueInstance`, `UMayDialogueNode_`, `UMayDialogueParticipant`.
3. Null dereference? → Check Instance lifetime (destroyed too early?).
4. Array out of bounds? → Check choice index validation (programmatic `SelectChoice` call).

When reporting a bug: provide repro steps + log excerpt (relevant lines only) + plugin version.

---

## Performance Issues

If the dialogue "stutters" or is noticeably slow:

| Symptom | What to check |
| --- | --- |
| Typewriter too fast or too slow | `TypewriterCharsPerSecond` in the speaker settings |
| Portrait texture is costly | Check texture size in asset details (max 512×512 recommended) |
| Babel generates many sounds at once | Reduce the number of BlipSounds samples |
| Camera blend stutters | Increase `BlendDuration`, reduce parallel blends |
