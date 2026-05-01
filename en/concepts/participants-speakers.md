---
description: Participant component on the actor vs. speaker definition in the asset — and how they find each other.
---

# Participants & Speakers

In MayDialogue there are two sides to a speaking character: the **participant component** on the actor in the level and the **speaker entry** in the dialogue asset. Both are connected via a tag.

## The concept at a glance

```text
Level (runtime)                  Dialogue Asset (authoring)
───────────────                  ──────────────────────────
GuardActor                       Speaker list
  └─ UMayDialogueParticipant       └─ "Guard"
       ParticipantTag:                  SpeakerTag: Dialogue.Speaker.Guard
       Dialogue.Speaker.Guard     ←──── DisplayName: "Guard"
       DisplayName: "Hans"              Portrait: TX_Guard_Neutral
       PersistentMemory: {...}          NodeColor: #CC3333
```

The component asks: *"Who is this in the level?"*
The speaker asks: *"How is he presented in **this** dialogue?"*

Why this separation? An NPC can have a different name and appearance in different dialogues. The same guard actor appears in the first dialogue as anonymous "Unknown Guard" (grey, no portrait) and after the reveal as "Hans, Captain" with his own portrait — because each asset defines its speaker entries freely.

> 📸 **Image placeholder:** `participant-vs-speaker-diagram.png` — Diagram: actor side on the left, asset side on the right, arrow via SpeakerTag.
> *Setup:* Two boxes side by side. Left: "GuardActor in level" with icons for ActorComponent and a tag symbol with text `Dialogue.Speaker.Guard`. Right: "DA_Guard_Greeting (Asset)" with a Speakers panel excerpt: entry "Guard", SpeakerTag field shows `Dialogue.Speaker.Guard`. A thick arrow connects both via the tag. Below: small text "Tag = bridge between level and asset".

## The participant component

You attach `UMayDialogueParticipant` via the Details panel to every dialogue-relevant actor — NPCs, the player pawn, objects with a voice.

### Key properties

| Property | Purpose |
| --- | --- |
| `ParticipantTag` | The match key. Convention: `Dialogue.Speaker.<Name>`. |
| `DisplayName` | Fallback name when the asset defines none of its own. |
| `Portrait` | Fallback portrait. Overridden by the asset speaker. |
| `DefaultDialogue` | The asset that starts on standard interaction. |
| `bAutoFacePartner` | Actor automatically turns to face the conversation partner. |
| `CameraTargetOffset` | Offset from the actor origin for CameraFocus nodes. |
| `PersistentMemory` | Variables that survive conversations (SaveGame-flagged). |

> 📸 **Image placeholder:** `participant-component-add.png` — Adding the Participant component to an actor.
> *Setup:* Details panel of a guard actor. "Add Component" button clicked, search field shows "MayDialogue", the entry `MayDialogueParticipant` is highlighted. No other content.

> 📸 **Image placeholder:** `participant-component-filled.png` — Fully filled-out Participant component.
> *Setup:* Details panel, section `MayDialogueParticipant`. Visible fields: `ParticipantTag = Dialogue.Speaker.Guard`, `DisplayName = Guard`, `DefaultDialogue = DA_Guard_Greeting` (asset reference), `bAutoFacePartner = true`, `CameraTargetOffset = (0, 0, 80)`. Portrait slot filled with Texture2D `TX_Guard_Neutral`.

### Three ways to start a dialogue

All three paths converge at the subsystem and produce the same instance.

**Blueprint — the easiest:**

> 📸 **Image placeholder:** `bp-start-default-dialogue.png` — Blueprint graph: StartDefaultDialogue on the component.
> *Setup:* BP graph of an interaction trigger (e.g. box overlap). `Event BeginOverlap` → `Get MayDialogueParticipant` (on GuardActor) → `Start Default Dialogue`. Pin `Instigator`: `Get Player Pawn`. All nodes connected and labeled.

```cpp
// C++ option 1 — on the component
Guard->FindComponentByClass<UMayDialogueParticipant>()->StartDefaultDialogue(Player);

// C++ option 2 — via the library (Blueprint-friendly)
UMayDialogueLibrary::StartDialogue(this, DA_Guard_Greeting, PlayerPawn, GuardActor);

// C++ option 3 — directly on the subsystem
UMayDialogueSubsystem::Get(this)->StartDialogue(DA_Guard_Greeting, PlayerPawn, GuardActor);
```

### Dynamic dialogue swapping

Use `SetActiveDialogue(NewAsset)` to change an NPC's default dialogue at runtime — for example when a quest has unlocked a new conversation state.

**Blueprint:**

> 📸 **Image placeholder:** `bp-set-active-dialogue.png` — Blueprint: SetActiveDialogue on the component after a quest event.
> *Setup:* BP graph of a QuestManager. Event `OnQuestCompleted` → `Get MayDialogueParticipant` (on GuardActor) → `Set Active Dialogue`. Pin `New Dialogue = DA_Guard_AfterRevelation`. Simple linear graph.

```cpp
Guard->FindComponentByClass<UMayDialogueParticipant>()->SetActiveDialogue(DA_Guard_AfterRevelation);
```

## The speaker entry in the asset

For each speaker in a dialogue asset you add an entry in the **Speakers panel**:

| Property | Purpose |
| --- | --- |
| `SpeakerTag` | Match key — must match the actor's `ParticipantTag`. |
| `DisplayName` | Name in the UI widget. Overrides the component fallback. |
| `Portrait` | Portrait in the UI widget. Overrides the component fallback. |
| `NodeColor` | Title bar color of all SayLine nodes for this speaker. |
| `AudioModeOverride` | Default / Spatial3D / Force2D. |
| `VolumeMultiplier` / `PitchMultiplier` | Per-speaker audio tuning. |

> 📸 **Image placeholder:** `speakers-panel-filled.png` — Speakers panel with two entries.
> *Setup:* Asset editor, Speakers panel tab open. Two entries: 1. "Guard" — SpeakerTag: `Dialogue.Speaker.Guard`, NodeColor: dark red, Portrait: TX_Guard_Neutral. 2. "Player" — SpeakerTag: `Dialogue.Speaker.Player`, NodeColor: dark blue, no portrait. Both entries expanded.

## Participant resolution at runtime

When a SayLine node targets the speaker tag `Dialogue.Speaker.Guard`, the plugin searches for a matching actor in this order:

1. Instigator actor
2. Target actor
3. All additionally registered participants (e.g. a third NPC in the conversation)
4. Fallback: no actor found → portrait and name come from the asset speaker entry, audio plays in 2D.

## PersistentMemory: What survives the dialogue

`PersistentMemory` is the place for variables that remain valid after the conversation — for example whether the player has already met this NPC, or how much trust has been built.

**Blueprint:**

> 📸 **Image placeholder:** `bp-persistent-memory.png` — Blueprint: SetPersistentBool on the Participant component.
> *Setup:* BP graph after a dialogue ends. `OnDialogueEnded` → `Get MayDialogueParticipant` → `Set Persistent Bool`. Pins: `Variable Name = "HasMet"`, `Value = true`. Simple graph, everything labeled.

```cpp
// Write
Guard->FindComponentByClass<UMayDialogueParticipant>()->SetPersistentBool("HasMet", true);
Guard->FindComponentByClass<UMayDialogueParticipant>()->SetPersistentInt("FriendshipPoints", 42);

// Read with default value
bool HasMet      = Part->GetPersistentBool("HasMet", false);
int32 Friendship = Part->GetPersistentInt("FriendshipPoints", 0);
```

`PersistentMemory` is flagged as `SaveGame`. Your own save game system will automatically include it when you serialize the component.

## Summary

- **Participant component** = the actor's identity in the level. Tag, memory, audio overrides, camera offset.
- **Speaker entry** = the presentation in the dialogue asset. Name, portrait, color, audio tuning.
- **Tag matching** connects both at runtime.
- `DefaultDialogue` and `SetActiveDialogue` control which conversation an NPC starts.
- `PersistentMemory` stores cross-conversation variables.

Next: [Variables & Scopes](variables-scopes.md).
