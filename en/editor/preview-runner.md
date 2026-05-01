---
description: Test a dialogue directly in the editor — without PIE, with GAS state simulation, culture switching, and live variables.
---

# Preview Runner

The Preview Runner plays a dialogue **directly inside the Asset Editor** — no level to load, no PIE to start, no player to spawn. Press Play, and the dialogue runs.

This is the tool where most dialogue work happens. Change a line, test it immediately, change it again — iteration time is under one second.

> 📸 **Image placeholder:** `preview-runner-active.png` — Preview panel with an active dialogue. Speaker portrait at the top, text with typewriter effect, two choice buttons visible.
> *Setup:* Asset `DA_Guard_Greeting` in the Preview Runner. Speaker "Guard" with portrait top-left, dialogue text "Halt! Who are you?" (half typed, typewriter effect). Below: two choice buttons: "A friend of the king." and "That's none of your business." Bottom: event log with "Entry → SayLine_Guard_01".

## Starting

1. Click the **Play Dialog** toolbar button — or the Play button directly in the Preview tab.
2. The runner starts at the Entry node.
3. The dialogue runs.

> 📸 **Image placeholder:** `preview-runner-start.png` — Toolbar with a red arrow pointing to the Play Dialog button, Preview tab next to it empty (before starting).
> *Setup:* Asset open, no preview active yet. Red arrow pointing to the Play Dialog button in the toolbar.

## Panel layout

The Preview panel is divided into three sections:

```
┌─────────────────────────────────────────────────────┐
│  [Culture: de]          [Stop]  [Advance]            │  ← Control bar
├─────────────────────────────────────────────────────┤
│                                                     │
│  [Portrait]  Guard                                  │  ← Dialogue display
│              "Halt! Who are you?"                   │
│                                                     │
│  [ A friend of the king. ]                          │
│  [ That's none of your business. ]                  │
├─────────────────────────────────────────────────────┤
│  Tags:  [ ] Story.Found.Codex   [x] Player.Alert    │  ← Preview state
│  Vars:  AngerLevel = 0  [Edit]                      │
│  Log:   Entry → SayLine_Guard_01                    │
└─────────────────────────────────────────────────────┘
```

## Control reference

| Button | Effect |
| --- | --- |
| **Play** | Start the preview |
| **Stop** | End the preview and reset state |
| **Advance** | In "waiting for click" mode: advance to the next node |
| **Skip Typewriter** | Show the full text immediately |
| **Choice buttons** | In PlayerChoice mode: make a selection |

## What works in the preview

| Feature | Behaviour |
| --- | --- |
| SayLine | Text with typewriter effect, speaker portrait visible |
| Voice assets | Played back in 2D |
| Babel synthesis | Triggered (with the active culture) |
| PlayerChoice | Clickable buttons |
| Branch | Evaluates requirements against the preview state |
| RandomLine | Picks randomly |
| Wait | Waits for an Advance click or timer |
| Link / SubGraph | Works as in production |
| Set Variable | Writes to the preview state |
| Fire Event | Logged (no actual Blueprint call) |
| Camera / Animation / Apply Effect | Logged, **not actually executed** |

## Simulating GAS state

The Preview Runner scans the tags, attributes, and abilities used in the asset and lists them in the Preview State section. You can **toggle them manually** before or while the dialogue runs:

- **Tags** — Checkbox list. Enabled tags simulate their presence on the player's ASC.
- **Attributes** — Number fields. Change the simulated value (e.g. set Health to 10).
- **Abilities** — Checkbox list. Simulates whether the player has an ability.

As soon as you change a value, requirement checks on Branch and Choice sub-nodes are re-evaluated live.

> 📸 **Image placeholder:** `preview-runner-gas-state.png` — Preview State section with a tag list and one active tag.
> *Setup:* Preview State expanded at the bottom. Tags section: `Story.Found.Codex` (checkbox unchecked), `Player.Alert` (checkbox checked, blue tick). Below: Attributes with an empty Health field = 100. Red arrow pointing to the tag checkboxes.

## Setting variables live

The Preview State section shows all dialogue-scope variables with their current value. Clicking a value opens an input field — change it and press Enter.

Useful for:
- Targeting specific branches directly (set AngerLevel to 3 without provoking the NPC three times)
- Testing branches that are rarely reached in a normal playthrough
- Overriding default values for the first test run

> 📸 **Image placeholder:** `preview-runner-variable-edit.png` — Preview State, variable "AngerLevel" in edit mode (input field active with value "3").
> *Setup:* Preview running or paused. Variable `AngerLevel` clicked, input field open with value "3". Red arrow pointing to the input field. The current SayLine text is visible in the dialogue display above.

## Culture switching

The culture dropdown at the top of the Preview panel switches the active language:

1. SayLine voice assets are re-resolved from the `VoicePerCulture` slot of the new culture.
2. `FText` values update automatically through UE's localisation system.
3. Babel synthesis is re-triggered with the new text.

> 📸 **Image placeholder:** `preview-runner-culture-switch.png` — Culture dropdown open with "de", "en", "fr" as options.
> *Setup:* Preview panel, culture dropdown expanded. Three options visible, "de" currently active. Red arrow pointing to the dropdown. The German text of the current SayLine is visible in the dialogue display below.

## Event log

At the bottom of the Preview State section, a time-ordered log shows:

- Node transitions: *"Entry → SayLine_Guard_01 → PlayerChoice_01"*
- Variable changes: *"AngerLevel: 0 → 1"*
- Fire Event triggers: *"Event: Dialogue.Event.QuestUpdated"*
- Errors: *"Requirement failed: CheckDialogueVariable(AngerLevel >= 3)"*

## When to use PIE instead

The Preview Runner is not suited for everything:

| Scenario | Better with |
| --- | --- |
| Dialogue requires real AI states | PIE + Debugger |
| Testing camera pans and animations | PIE |
| GameplayEffects must take real effect | PIE |
| Testing multiplayer paths | PIE |

For everything else — text, choices, variables, audio — the Preview Runner is faster.

Next: [Debugger & Breakpoints →](debugger.md)
