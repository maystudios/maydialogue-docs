---
description: A clear cause and a concrete solution for each symptom.
---

# Common Issues

Each problem is phrased as a question so you can quickly find what's blocking you right now.

---

## Dialogue Doesn't Start

**Symptom:** You call `StartDialogue` (e.g. in an interaction trigger), but nothing happens. No widget, no log entry, nothing.

**Common causes and solutions:**

<details>
<summary>Asset not compiled</summary>

The asset contains unsaved compile errors. Open the dialogue asset, click **Compile**, and fix all validator errors in the Compiler Results tab.

{% hint style="warning" %}
An asset with a compile error is rejected by the Subsystem before an Instance is created.
{% endhint %}

</details>

<details>
<summary>Entry node missing</summary>

Every dialogue asset needs exactly one **Entry** node. If it's missing, the compiler outputs `Asset has no entry point`. Add the Entry node via the palette and connect it to the first SayLine or Branch node.

</details>

<details>
<summary>Participant component missing or tag doesn't match</summary>

Every actor involved in the dialogue needs a `UMayDialogueParticipant` component with the correct `ParticipantTag`.

- Player Pawn: `ParticipantTag = Dialogue.Speaker.Player`
- NPC: `ParticipantTag` must exactly match the SpeakerTag used in the SayLines.

Open the actor in the Details panel, locate the component, and verify the tag value.

</details>

<details>
<summary>Another dialogue is already running</summary>

The Subsystem allows only one active dialogue by default. Check with `MayDialogueSubsystem → IsAnyDialogueActive()`. Stop any running dialogues first with `StopAllDialogues()`.

</details>

> 📸 **Image placeholder:** `common-issues-start-participant.png` — Participant component in the Details panel of an NPC actor.
> *Setup:* NPC Blueprint open in the editor. In the Components panel, the `UMayDialogueParticipant` component is selected. The Details panel on the right shows: `ParticipantTag` field with value `Dialogue.Speaker.Guard`. The `DefaultDialogue` asset slot shows an assigned dialogue asset.

---

## Widget Doesn't Appear

**Symptom:** The Output Log shows that the dialogue has started (`StartDialogue: Instance created`), but no dialogue box appears in the game or PIE.

<details>
<summary>No widget setup in Project Settings</summary>

Go to **Edit → Project Settings → MayDialogue → UI**. Check:

- `DefaultDialogueWidgetClass` is filled in (your UMG class or the included default widget).
- Alternatively: `bUseSlateDialogueWidget = true` for the built-in Slate fallback.

Without one of these options, the plugin doesn't know what to display.

</details>

<details>
<summary>UMG widget has wrong BindWidget names</summary>

Open your UMG widget in the Designer. Critical mandatory slots are:

| Slot Name | Type |
| --- | --- |
| `DialogFrameWidget` | Border / Overlay |
| `TextWidget` | RichTextBlock |
| `ChoiceListWidget` | UMayDialogueWidget_ChoiceList |

A wrong or missing name produces a silent warning — no crash, but no visible widget.

</details>

<details>
<summary>Widget not rebound after level change (known issue)</summary>

The static widget doesn't always survive level teardown cleanly. **Workaround:** Call `Subsystem → StopAllDialogues()` on level change (e.g. in the Level Blueprint `BeginPlay` or in `GameMode::HandleSeamlessTravel`) so the widget re-registers correctly.

</details>

> 📸 **Image placeholder:** `common-issues-widget-settings.png` — MayDialogue UI settings in Project Settings.
> *Setup:* Editor → Edit → Project Settings → MayDialogue category → UI section. Visible: `DefaultDialogueWidgetClass` dropdown with selected class `WBP_DialogueHUD`, `bUseSlateDialogueWidget` checkbox (unchecked).

---

## Choices Missing

**Symptom:** A PlayerChoice node is active, but no answer buttons appear in the game — or only some of them do.

<details>
<summary>All choices are FailedAndHidden</summary>

When every Choice in a PlayerChoice node has a Requirement that fails **and** the Requirement is configured as `FailedAndHidden`, all buttons disappear.

Test in the **Preview Runner**: set tags manually to simulate Requirements. If the choices appear there, the GAS tag state in PIE is the problem.

</details>

<details>
<summary>ChoiceButtonClass not set</summary>

In the `UMayDialogueWidget_ChoiceList` widget (Blueprint), the `ChoiceButtonClass` property must point to a valid button class. If it's empty, no buttons are created.

</details>

<details>
<summary>ChoiceListWidget slot missing in the top-level widget</summary>

The parent dialogue widget must have a `ChoiceListWidget` bind slot. If it's missing, the plugin falls back to a legacy path that doesn't render choices.

</details>

> 📸 **Image placeholder:** `common-issues-choices-preview.png` — PlayerChoice node in the Preview Runner with visible choice buttons.
> *Setup:* Open a dialogue asset with a PlayerChoice node, start the Preview Runner. Visible: the preview panel shows an NPC SayLine and two choice buttons below it. The first choice has a green lock icon (requirement met), the second a red lock icon (FailedButVisible).

---

## Audio Missing

**Symptom:** The SayLine appears as text, but no voice is audible.

<details>
<summary>Voice asset for the current culture is missing</summary>

Check the `DialogueVoice` slot in the SayLine Details panel. It is split by culture key. If your active culture has no assigned audio asset, the SayLine stays silent.

</details>

<details>
<summary>AudioComponent is obstructed or attenuation is too aggressive</summary>

The speaker actor plays the sound spatially. If the player is too far away or the attenuation curve is set too aggressively, nothing is audible. Check the `AttenuationOverride` on the Participant component, or set `AudioModeOverride = Force2D` on the speaker for close-range dialogue.

</details>

<details>
<summary>SoundClass is muted</summary>

Check under **Edit → Project Settings → Audio** or in the mixer whether the `SoundClass` of the dialogue asset (e.g. `SC_Voice`) is at full volume.

</details>

> 📸 **Image placeholder:** `common-issues-audio-sayline.png` — SayLine node with a filled voice asset slot in the Details panel.
> *Setup:* Select SayLine node in the dialogue asset. Details panel shows: `SpeakerTag = Dialogue.Speaker.Guard`, `DialogueVoice` array with one entry (Key: `en`, Value: assigned `SoundWave` asset). `AdvanceModeOverride = AfterVoice` selected.

---

## Babel Silent

**Symptom:** The SayLine has no voice asset, but the procedural Babel voice is still not audible.

<details>
<summary>Babel not enabled</summary>

Go to **Project Settings → MayDialogue → Audio**. `bEnableBabelVoice` must be `true`.

</details>

<details>
<summary>No BabelProfile assigned</summary>

The speaker or global settings must have a `DefaultBabelProfile` set. Without a profile, Babel doesn't know what sound to generate.

</details>

<details>
<summary>BlipSounds empty and ProceduralDefaults disabled</summary>

If `bUseProceduralDefaults = false` and no `BlipSounds` array is filled, Babel produces no sound. Either assign samples or set `bUseProceduralDefaults = true`.

</details>

<details>
<summary>OnCharacterRevealed not connected</summary>

In the Blueprint widget, the `OnCharacterRevealed` event from the `TextWidget` must be connected to `BabelSynth → OnCharacterRevealed`. Without this connection, Babel receives no signal to play.

</details>

---

## Variable Not Persistent

**Symptom:** You set a variable with a SetVariable node, but a subsequent Requirement check or a second dialogue reads the old value.

<details>
<summary>Wrong scope</summary>

Dialogue-scope variables only exist while the current dialogue is running. If you need a variable across multiple dialogues, use **Participant scope**.

| Scope | Lifetime |
| --- | --- |
| Dialogue scope | Only while this dialogue Instance is running |
| Participant scope | As long as the actor exists in the level (optionally SaveGame) |

</details>

<details>
<summary>SetVariable currently only writes Dialogue scope (known issue #13)</summary>

The SetVariable node currently supports only Dialogue scope. **Workaround:** Use a Blueprint SideEffect node that calls `Participant → SetPersistentVariable` directly.

</details>

<details>
<summary>Variable not declared in the Variables panel</summary>

Open the Variables panel of the dialogue asset. The variable must be created there with the correct name and type before any node can reference it.

</details>

<details>
<summary>Type mismatch</summary>

The validator warns about inconsistent types (e.g. a Bool variable with a String setter). Check the Compiler Results tab after compiling.

</details>

> 📸 **Image placeholder:** `common-issues-variables-panel.png` — Variables panel of a dialogue asset with several declared variables.
> *Setup:* Open dialogue asset, activate the Variables tab. Visible: table with columns Name, Type, Scope, Default Value. Example entries: `bMetGuard` (Bool, Participant), `ChosenPassword` (String, Dialogue).

---

## Dialogue Gets Stuck

**Symptom:** The dialogue starts, shows a SayLine, and then nothing happens. No advance, no end.

<details>
<summary>AdvanceMode = AfterVoice, but no voice asset</summary>

The dialogue waits for the voice end callback, which never arrives because no audio asset is set. Solution: assign a voice asset, or change `AdvanceModeOverride` to `Manual` or `Timer`.

</details>

<details>
<summary>Wait node waiting for an event that never fires</summary>

A Wait node with a `WaitEventTag` blocks until that tag event is received by the Subsystem. Check whether the event is actually being fired somewhere in Blueprint (`Subsystem → FireDialogueEvent(Tag)`).

</details>

<details>
<summary>PlayAnimation with bWaitForMontageEnd = true, montage doesn't exist</summary>

If the montage asset is empty or invalid, the end delegate is never fired. Set `bWaitForMontageEnd = false` or assign a valid montage asset.

</details>

**Diagnose with the debugger:** Set a breakpoint on the node before the problem, start PIE, trigger the dialogue. When the breakpoint hits, click **Step Over** and watch which node doesn't advance.

> 📸 **Image placeholder:** `common-issues-debugger-breakpoint.png` — Dialogue debugger in the editor with an active breakpoint on a SayLine node.
> *Setup:* Open dialogue asset, activate the Debugger tab. Set a red breakpoint dot on a SayLine node (right-click → Toggle Breakpoint). PIE active, dialogue triggered. Visible: the node lights up orange/yellow as the currently active node, the Debugger tab shows the current variable values.

---

## Compile Error

**Symptom:** The Compiler Results tab shows an error, but clicking the error jumps to a wrong or unclear node.

<details>
<summary>Other assets have unsaved changes with cross-references</summary>

Save all open dialogue assets (`Ctrl+Shift+S` for all) and compile again.

</details>

<details>
<summary>Editor restart needed after plugin changes</summary>

After major changes to plugin code (e.g. a new node class), all assets need to be recompiled. Restart the editor and reopen the asset.

</details>

---

## Sub-Graph Return Missing

**Symptom:** A Sub-Graph is entered and reaches its Exit node, but instead of returning to the calling graph, the entire dialogue ends.

<details>
<summary>bReturnAfterExit not set</summary>

The SubGraph node in the parent graph has a `bReturnAfterExit` property. It must be `true`. The default value should be `true` — check whether it was accidentally disabled.

</details>

<details>
<summary>SubGraph entry missing in the sub-asset</summary>

If the called asset has no Entry node, the validator outputs a warning. The return jump cannot be resolved cleanly. Add the Entry node and connect it.

</details>

---

## "Other MayDialogue Participant is null"

**Symptom:** `StartDefaultDialogue(Other)` fails with a log warning about an empty `Other` parameter.

The `Other` actor has no `UMayDialogueParticipant` component, or it hasn't been created yet (e.g. for dynamically spawned actors).

**Solution:** Add the component as a default component in the Blueprint Defaults panel of the actor. For dynamically spawned actors: use `AddComponentByClass<UMayDialogueParticipant>` in `BeginPlay`.

> 📸 **Image placeholder:** `common-issues-participant-beginplay.png` — Blueprint graph with AddComponent call in BeginPlay.
> *Setup:* Open NPC Blueprint, Event Graph → BeginPlay. Visible: `Event BeginPlay` → `Add Component by Class` (Component Class: `UMayDialogueParticipant`) → `Set Participant Tag` (Tag: `Dialogue.Speaker.Guard`). All pins fully connected.
