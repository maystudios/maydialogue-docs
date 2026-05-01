---
description: All enums and important structs of the plugin — values, fields, one-sentence explanations.
---

# Types & Enums (Reference)

All public enums and structs in one table. For deeper explanations, each group links to the relevant chapter.

---

## Enums

### EMayDialogueStatus

Lifecycle state of a running Instance. Readable directly as `Instance->Status`.

| Value | Meaning |
|---|---|
| `Inactive` | Initial state, not yet started. |
| `Active` | Running, not waiting. |
| `WaitingForAdvance` | Waiting for player input or timer (SayLine). |
| `WaitingForChoice` | PlayerChoice was presented, player must choose. |
| `Completed` | Exit node with Completed status reached. |
| `Aborted` | Externally interrupted. |

---

### EMayDialogueExitStatus

Result of a dialogue ending. Parameter in `OnDialogueEnded`.

| Value | Meaning |
|---|---|
| `Completed` | Cleanly ended — Exit node reached. |
| `Failed` | Exit node with Failed status or Requirement abort. |
| `Aborted` | Externally interrupted (new dialogue, level travel, `StopAllDialogues`). |

---

### EMayDialogueAdvanceMode

Controls when a SayLine moves to the next node.

| Value | Meaning |
|---|---|
| `Manual` | Player input (click / key press). |
| `Timer` | After `AutoAdvanceDelay` seconds. |
| `AfterVoice` | After voice playback ends (fallback: typewriter end). |
| `AfterAnimation` | After the current montage on the speaker ends. |
| `Immediate` | No waiting, advance immediately (for pure logic nodes). |

---

### EMayDialogueVariableType

Types allowed in dialogue and participant variables.

| Value | C++ Type | String Format |
|---|---|---|
| `Bool` | `bool` | `"true"` / `"false"` |
| `Int` | `int32` | `"42"` |
| `Float` | `float` | `"3.14"` |
| `String` | `FString` | arbitrary |
| `Tag` | `FGameplayTag` | `"Dialogue.Mood.Friendly"` |

---

### EMayDialogueVariableScope

Where a variable is stored.

| Value | Meaning | Lifetime |
|---|---|---|
| `Dialogue` | Only for the lifetime of the Instance. | Ends with the dialogue. |
| `Participant` | In the `PersistentMemory` of the Participant component. | Survives conversations (SaveGame field). |

---

### EMayDialogueRequirementResult

Result of a Requirement evaluation. Determines how the choice filter responds.

| Value | Meaning |
|---|---|
| `Passed` | Requirement met — Choice available. |
| `FailedButVisible` | Not met, Choice is displayed greyed out. |
| `FailedAndHidden` | Not met, Choice is completely hidden. |

---

### EMayDialogueNodeFailBehavior

What happens when a node's Requirements are not met.

| Value | Meaning |
|---|---|
| `Skip` | Skip the node, continue at the output pin. |
| `Abort` | End the dialogue with `EMayDialogueExitStatus::Failed`. |

---

### EMayDialogueAudioMode

Spatial mode for voice playback.

| Value | Meaning |
|---|---|
| `Default` | Project default (`bForce2D` from Project Settings). |
| `Spatial3D` | 3D-positioned at the speaker actor. |
| `Force2D` | Always 2D, ignores the project default. |

---

### EMayDialogueBabelMode

When Babel voice synthesis kicks in.

| Value | Meaning |
|---|---|
| `Off` | Never. |
| `FallbackOnly` | Only when no voice asset is present. |
| `Always` | Always — even when a voice asset is set (preview mode). |

---

### EMayDialogueBabelSyncMode

How Babel synchronizes with the typewriter.

| Value | Meaning |
|---|---|
| `PerCharacter` | One sound per typewriter character. |
| `PerWord` | One Babel blip per word. |
| `Continuous` | Continuous tone, independent of the typewriter tick. |

---

## Structs

### FMayDialogueMessage

The finished message structure delivered to the UI via `OnMessageReceived`.

```cpp
USTRUCT()
struct FMayDialogueMessage
{
    FGameplayTag          SpeakerTag;       // Which speaker
    FText                 DisplayName;      // Display name
    FText                 Text;             // Dialogue text
    USoundBase*           Voice;            // Voice asset (nullptr if none)
    FGameplayTagContainer EmotionTags;      // Emotion context
    EMayDialogueAdvanceMode AdvanceMode;    // How to advance after this line
    float                 AutoAdvanceDelay; // Delay for AdvanceMode::Timer
};
```

---

### FMayDialogueChoiceEntry

A single choice entry after Requirement evaluation and filtering.

```cpp
USTRUCT()
struct FMayDialogueChoiceEntry
{
    int32                         ChoiceIndex;        // 0-based index for SelectChoice()
    FText                         Text;               // Display text of the choice
    FGameplayTagContainer         Tags;               // Choice tags (for analytics, styling)
    bool                          bAvailable;         // true = selectable
    FText                         UnavailableReason;  // Reason when not selectable (for greyed UI)
    FGuid                         TargetNodeGuid;     // Target node when selected
    EMayDialogueRequirementResult RequirementResult;  // Passed / FailedButVisible / FailedAndHidden
};
```

---

### FMayDialogueSpeaker

Entry in the Speakers panel of the dialogue asset editor.

```cpp
USTRUCT()
struct FMayDialogueSpeaker
{
    FGameplayTag SpeakerTag;                               // Unique speaker identifier
    FText        DisplayName;                              // Display name in the UI
    TSoftObjectPtr<UTexture2D> Portrait;                   // Portrait texture
    FGameplayTagContainer DefaultEmotionTags;              // Default emotions
    TSoftObjectPtr<USoundClass> VoiceSoundClassOverride;   // Sound class override
    TSoftObjectPtr<UMayDialogueBabelProfile> BabelProfileOverride; // Babel profile override
};
```

---

### FMayDialogueContext

Passed to all `ExecuteNode` calls. Aggregates everything a node needs for world access.

```cpp
USTRUCT()
struct FMayDialogueContext
{
    UMayDialogueInstance*    Instance;          // The running Instance
    AActor*                  Instigator;        // Who started the dialogue
    AActor*                  Target;            // Who is being spoken to
    UMayDialogueParticipant* Speaker;           // Active speaker (Participant component)
    FGameplayTag             CurrentSpeakerTag; // Tag of the active speaker
};
```

---

### FMayDialogueParticipants

Container on the Instance for participant lookup by tag.

```cpp
USTRUCT()
struct FMayDialogueParticipants
{
    TMap<FGameplayTag, TWeakObjectPtr<UMayDialogueParticipant>> ByTag;
};
```

---

### FMayDialogueScopeEntry

A frame on the internal scope stack (for Link and SubGraph nodes).

```cpp
USTRUCT()
struct FMayDialogueScopeEntry
{
    UMayDialogueAsset* Asset;          // Asset of the sub-dialogue
    FGuid              ReturnNodeGuid; // Where to return after the sub-dialogue
};
```

---

### FMayDialogueBranchPoint

A single branch in a Branch node.

```cpp
USTRUCT()
struct FMayDialogueBranchPoint
{
    FText Description;                           // Editorial label
    TArray<UMayDialogueRequirement*> Requirements; // Conditions (all must return Passed)
    FGuid TargetNodeGuid;                        // Target node when conditions are met
};
```

The order of `BranchPoints` determines evaluation priority (first match wins).

---

## See Also

- [Runtime → Read/Write API](../runtime/read-write-api.md) — `EMayDialogueVariableType/Scope` in practice.
- [API: Delegates](api-delegates.md) — how `FMayDialogueMessage` and `FMayDialogueChoiceEntry` appear in delegate parameters.
- [Concepts → Instance & Lifecycle](../concepts/instance-lifecycle.md) — how the status enums are traversed.
- [GAS → Requirements](../gas/requirements.md) — how `EMayDialogueRequirementResult` is evaluated.
