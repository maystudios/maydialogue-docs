---
description: SaveGame integration at a glance.
---

# Persistence

Some dialogue data needs to survive beyond a session: whether the player has already met an NPC, how many times they have spoken to them, which conversation variant they chose. MayDialogue stores this in `PersistentMemory` on the Participant component.

## Philosophy

MayDialogue does not ship its own SaveGame system. Instead:

* `UMayDialogueParticipant::PersistentMemory` is marked with `UPROPERTY(SaveGame)` — that is enough for UE's standard serialisation to include it.
* If you do not have your own SaveGame system, the built-in [QuickSave Helper](quicksave-helper.md) is available as a starting point.

## What is saved — and what is not

| Saved | Not saved |
| --- | --- |
| `PersistentMemory` of all participants | Currently running dialogue instance |
| Global `GlobalMemory` (via `UMayDialogueSaveGame`) | Dialogue-scope variables of the running instance |
| | UI state |

{% hint style="info" %}
**Dialogue scope is temporary by definition.** Variables you create inside a dialogue (Scope = Dialogue) exist only for the duration of the conversation. If you want to remember something permanently, write it to `PersistentMemory` (Scope = Participant).
{% endhint %}

## Overview: how does everything connect?

```text
Participant A  ──PersistentMemory──┐
Participant B  ──PersistentMemory──┼──► UMayDialogueSaveGame ──► SaveGame slot
Participant C  ──PersistentMemory──┘        + GlobalMemory
```

> 📸 **Image placeholder:** `persistence-overview-diagram.png` — Flow diagram: three Participant components feed into UMayDialogueSaveGame, which is written to a slot.
> *Setup:* Simple block diagram graphic (no UE editor needed). Three boxes on the left ("Participant Guard", "Participant Merchant", "Participant Player") with arrows pointing right to "UMayDialogueSaveGame" (centre), then an arrow pointing right to "SaveGame Slot AutoSave". "GlobalMemory" label below UMayDialogueSaveGame.

## Chapters

* [SaveGame Integration](save-integration.md) — Integrating with your own SaveGame system.
* [Participant Memory](participant-memory.md) — What PersistentMemory can do, getters/setters, events.
* [QuickSave Helper](quicksave-helper.md) — The built-in helper for projects without their own SaveGame system.

> 📸 **Image placeholder:** `persistence-participant-component.png` — Participant component in the Details panel of an NPC actor.
> *Setup:* Select an NPC actor in the Level Editor. Details panel shows the `UMayDialogueParticipant` component. Expanded to show: `ParticipantTag`, `DisplayName`, `DefaultDialogue`, then `PersistentMemory` (greyed out / empty, no values set yet). Category label "Persistence" visible.

> 📸 **Image placeholder:** `persistence-scope-comparison.png` — Two variable panels side by side: Dialogue scope vs. Participant scope.
> *Setup:* MayDialogue Editor open. Variables panel shows two groups: "Dialogue Scope" (variables `DialogueChoice`, `Temp`) and "Participant Scope" (variables `HasMet`, `MeetingCount`). Participant-scope variables are marked with a save icon.
