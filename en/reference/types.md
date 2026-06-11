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
| `Aborted` | Externally interrupted (new dialogue, level travel, `AbortAllDialogues`). |

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

Controls the Babel synthesis algorithm per profile.

| Value | Meaning |
|---|---|
| `BlipPerCharacter` | A short blip sound per revealed character ("bip bip bip"). Classic VN style. |
| `PhonemeBase` | Sinusoidal tones based on vowel/consonant type. Abstract or creature voices. |

---

### EMayDialogueBabelSyncMode

How Babel synchronises with the typewriter.

| Value | Meaning |
|---|---|
| `TypewriterSync` | Reacts to every letter revealed by the typewriter. Precise per-character synchronisation. |
| `Continuous` | Internal timer, independent of the typewriter tick. For creature growls or non-typewriter flows. |

---

### EMayDialogueBabelEngine

Selects the synthesis back-end used by `UMayDialogueBabelSynth`. Configured on the `UMayDialogueBabelProfile` asset.

| Value | Meaning |
|---|---|
| `Granular` | **Default.** Plays pre-recorded blip samples from the profile's sample pools with random pitch/volume jitter. Fears-to-Fathom / Animal Crossing quality. |
| `Biquad` | Legacy procedural formant synthesiser (RBJ bandpass filter chain). Retained as opt-in for projects that relied on the original DSP path. DisplayName: *Biquad Synthesizer (Legacy)*. |

---

## Structs

### FMayDialogueMessage

The finished message structure delivered to the UI via `OnMessageReceived`.

```cpp
USTRUCT(BlueprintType)
struct FMayDialogueMessage
{
    FGameplayTag               SpeakerTag;         // Which speaker
    FText                      SpeakerDisplayName; // Display name in the UI
    FText                      Text;               // Dialogue text
    TSoftObjectPtr<UTexture2D> SpeakerPortrait;    // Portrait (async-loaded by the widget)
    TSoftObjectPtr<USoundBase> Voice;              // Voice asset (soft ref; IsNull() if none)
    FGameplayTagContainer      EmotionTags;        // Emotion context
    EMayDialogueAdvanceMode    AdvanceMode;        // How to advance after this line
    float                      AutoAdvanceDelay;   // Delay for AdvanceMode::Timer
};
```

`Voice` and `SpeakerPortrait` are **soft references** (`TSoftObjectPtr`) so they survive a net RPC even when not yet loaded on the receiving client — see [Multiplayer](../runtime/multiplayer.md) for the streaming detail.

---

### FMayDialogueChoiceEntry

A single choice entry after Requirement evaluation and filtering.

```cpp
USTRUCT(BlueprintType)
struct FMayDialogueChoiceEntry
{
    FText                         ChoiceText;         // Display text of the choice
    FGameplayTagContainer         ChoiceTags;         // Choice tags (for analytics, styling)
    EMayDialogueRequirementResult AvailabilityStatus; // Passed / FailedButVisible / FailedAndHidden
    FText                         UnavailableReason;  // Reason when not selectable (for greyed UI)
    int32                         ChoiceIndex;        // Authored index in the full choice list
    FGuid                         TargetNodeGuid;     // Target node when selected
    float                         Weight;             // Weighted-random default-on-timeout weight
};
```

Helpers `IsAvailable()` (status == `Passed`) and `IsVisible()` (status != `FailedAndHidden`) read `AvailabilityStatus`.

{% hint style="warning" %}
`ChoiceIndex` is the **authored** index in the node's full choice list. `SelectChoice` / `ServerSelectChoice` expect the **position** of the entry within the *presented visible* list, which differs when a hidden choice precedes it. Pass the entry's position in the received array, not `ChoiceIndex`, unless you know no choice is hidden.
{% endhint %}

---

### FMayDialogueSpeaker

Entry in the Speakers panel of the dialogue asset editor.

```cpp
USTRUCT()
struct FMayDialogueSpeaker
{
    FGameplayTag SpeakerTag;                                  // Unique speaker identifier
    FText        DisplayName;                                 // Display name in the UI
    TSoftObjectPtr<UTexture2D> Portrait;                      // Portrait texture
    FLinearColor NodeColor;                                   // Editor-only graph tint
    EMayDialogueAudioMode AudioModeOverride;                  // Per-speaker 2D/3D override
    TSoftObjectPtr<USoundClass> SoundClassOverride;           // Sound class override
    TSoftObjectPtr<USoundAttenuation> AttenuationOverride;    // 3D attenuation override
    float VolumeMultiplier;                                   // Per-speaker volume scalar (default 1.0)
    float PitchMultiplier;                                    // Per-speaker pitch scalar (default 1.0)
    TSoftObjectPtr<UMayDialogueBabelProfile> BabelProfile;    // Per-speaker Babel profile
};
```

---

### FMayDialogueContext

Passed to all `ExecuteNode` calls. Aggregates everything a node needs for world access.

```cpp
USTRUCT(BlueprintType)
struct FMayDialogueContext
{
    TWeakObjectPtr<UMayDialogueInstance> DialogueInstance;       // The running Instance (weak ref)
    TObjectPtr<AActor>                   Instigator;             // Who started the dialogue
    TObjectPtr<AActor>                   Target;                 // Who is being spoken to
    FGuid                                CurrentNodeGuid;        // Node currently being entered (mid-traversal)
    bool                                 bOwningNodeReachCounted; // True when the current reach is already counted

    UObject* ResolveInstigatorASC() const; // Instigator's ASC, or nullptr
    UWorld*  GetWorld() const;
};
```

---

### FMayDialogueParticipants

Container on the Instance for participant lookup by tag.

```cpp
USTRUCT()
struct FMayDialogueParticipants
{
    TMap<FGameplayTag, TWeakObjectPtr<UMayDialogueParticipant>> ParticipantMap;
    // Use the helpers, not the map directly:
    UMayDialogueParticipant*          FindParticipant(const FGameplayTag& Tag) const;
    TArray<UMayDialogueParticipant*>  GetAllParticipants() const;
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

A grouping struct used internally to carry a set of choice entries.

```cpp
USTRUCT(BlueprintType)
struct FMayDialogueBranchPoint
{
    TArray<FMayDialogueChoiceEntry> Choices;
};
```

{% hint style="info" %}
Branch *routing* itself lives on the `UMayDialogueNode_Branch` node: it evaluates a single `Condition` requirement and takes the **True** output when it passes, the **False** output when it fails, and an optional **Default** output when `bHasFallback` is set — not on this struct. See [Branch Node](../nodes/core/branch.md).
{% endhint %}

---

## See Also

- [Runtime → Read/Write API](../runtime/read-write-api.md) — `EMayDialogueVariableType/Scope` in practice.
- [API: Delegates](api-delegates.md) — how `FMayDialogueMessage` and `FMayDialogueChoiceEntry` appear in delegate parameters.
- [Concepts → Instance & Lifecycle](../concepts/instance-lifecycle.md) — how the status enums are traversed.
- [GAS → Requirements](../gas/requirements.md) — how `EMayDialogueRequirementResult` is evaluated.
