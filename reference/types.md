# Typen & Enums (Referenz)

Alle öffentlichen Enums und Structs des Plugins in einer Tabelle. Jeder Eintrag hat eine Ein-Satz-Erklärung. Für tiefergehende Erklärungen verlinkt jede Gruppe auf das zugehörige Kapitel.

## Enums

### EMayDialogueStatus

Lifecycle-State einer Instance. Siehe [Instance & Lifecycle](../concepts/instance-lifecycle.md).

| Wert | Bedeutung |
| --- | --- |
| `Inactive` | Initialer Zustand, noch nicht gestartet. |
| `Active` | Läuft, kein Warten. |
| `WaitingForAdvance` | Auf Spieler-Advance oder Timer wartend. |
| `WaitingForChoice` | PlayerChoice wurde präsentiert. |
| `Completed` | Exit mit Status Completed erreicht. |
| `Aborted` | Abort durch Code oder Requirement-Fail. |

### EMayDialogueAdvanceMode

Steuert, wann eine SayLine zum nächsten Node weitergeht.

| Wert | Bedeutung |
| --- | --- |
| `Manual` | Spieler-Input (Klick / Tastendruck). |
| `Timer` | Nach `AutoAdvanceDelay`-Sekunden. |
| `AfterVoice` | Nach Voice-Wiedergabe-Ende (Fallback: Typewriter-Ende). |
| `AfterAnimation` | Nach laufender Animation (Montage-End). |
| `Immediate` | Kein Warten, sofort weiter (nur für Logik-Durchläufe). |

### EMayDialogueRequirementResult

Ergebnis einer Requirement-Evaluation. Siehe [GAS → Requirements](../gas/requirements.md).

| Wert | Bedeutung |
| --- | --- |
| `Passed` | Requirement erfüllt. |
| `FailedButVisible` | Nicht erfüllt, UI zeigt die Option greyed an. |
| `FailedAndHidden` | Nicht erfüllt, UI filtert die Option komplett raus. |

### EMayDialogueVariableType

Typen, die in Dialogue- und Participant-Variablen erlaubt sind.

| Wert | Bedeutung |
| --- | --- |
| `Bool` | `bool`. |
| `Int` | `int32`. |
| `Float` | `float`. |
| `String` | `FString`. |
| `Tag` | `FGameplayTag`. |

### EMayDialogueVariableScope

Wo eine Variable gespeichert wird.

| Wert | Bedeutung |
| --- | --- |
| `Dialogue` | Nur für die Lebensdauer der Instance. |
| `Participant` | Im PersistentMemory des Participants (überdauert Gespräche). |

### EMayDialogueExitStatus

Ergebnis eines Dialog-Endes.

| Wert | Bedeutung |
| --- | --- |
| `Completed` | Sauber beendet (Exit-Node reached). |
| `Failed` | Exit-Node mit `Failed`-Status oder Requirement-Abort. |
| `Aborted` | Extern unterbrochen (neuer Dialog, Level-Travel, `StopAllDialogues`). |

### EMayDialogueNodeFailBehavior

Was passiert, wenn ein Node seine Requirements nicht erfüllt.

| Wert | Bedeutung |
| --- | --- |
| `Skip` | Node überspringen, mit Output-Pin weiter. |
| `Abort` | Dialog mit `EMayDialogueExitStatus::Failed` beenden. |

### EMayDialogueAudioMode

Spatial-Mode für Voice-Wiedergabe.

| Wert | Bedeutung |
| --- | --- |
| `Default` | Projekt-Default (siehe `bForce2D` in Projekt-Settings). |
| `Spatial3D` | 3D-Positioniert am Speaker-Actor. |
| `Force2D` | Unabhängig vom Default als 2D abspielen. |

### EMayDialogueBabelMode

Wann soll Babel-Synthese einspringen? Siehe [Audio → Babel-System](../audio/babel-system.md).

| Wert | Bedeutung |
| --- | --- |
| `Off` | Nie. |
| `FallbackOnly` | Nur wenn kein Voice-Asset vorhanden. |
| `Always` | Immer, auch wenn Voice-Asset gesetzt ist (Preview-Modus). |

### EMayDialogueBabelSyncMode

Wie synchronisiert Babel mit dem Typewriter?

| Wert | Bedeutung |
| --- | --- |
| `PerCharacter` | Ein Pling/Laut pro Typewriter-Zeichen. |
| `PerWord` | Pro Wort einen Babel-Block. |
| `Continuous` | Kontinuierlicher Ton, unabhängig vom Typewriter-Takt. |

### EMayDialogueTaskResultType

Diskriminator in `FMayDialogueTaskResult`. Details in [Instance & Lifecycle → Node-Ausführung](../concepts/instance-lifecycle.md#node-ausführung).

| Wert | Bedeutung |
| --- | --- |
| `AdvanceDialogue` | Weiter zum angegebenen Next-Node. |
| `AdvanceDialogueWithChoice` | Semantischer Marker nach Choice-Selektion. |
| `PauseDialogueAndPresentChoices` | Setzt `WaitingForChoice`. |
| `ReturnToLastChoice` | Springt zum letzten PlayerChoice zurück. |
| `ReturnToCurrentChoice` | Wiederholt die aktuelle Choice-Präsentation. |
| `ReturnToDialogueStart` | Springt an den Entry. |
| `AbortDialogue` | Beendet als `Aborted`. |

## Structs

### FMayDialogueTaskResult

```cpp
USTRUCT()
struct FMayDialogueTaskResult
{
    EMayDialogueTaskResultType Type;
    FGuid                      NextNodeGuid;
};
```

Der Rückgabewert von `ExecuteNode`. Static-Helper: `Advance(Guid)`, `Abort()`, `PauseAndPresentChoices()`, `ReturnToStart()`, `ReturnToLast()`, `ReturnToCurrent()`.

### FMayDialogueSpeaker

```cpp
USTRUCT()
struct FMayDialogueSpeaker
{
    FGameplayTag SpeakerTag;
    FText        DisplayName;
    TSoftObjectPtr<UTexture2D> Portrait;
    FGameplayTagContainer DefaultEmotionTags;
    TSoftObjectPtr<USoundClass> VoiceSoundClassOverride;
    TSoftObjectPtr<UMayDialogueBabelProfile> BabelProfileOverride;
};
```

Eintrag im Speakers-Panel des Asset-Editors. Siehe [Participants & Sprecher](../concepts/participants-speakers.md).

### FMayDialogueMessage

```cpp
USTRUCT()
struct FMayDialogueMessage
{
    FGameplayTag          SpeakerTag;
    FText                 DisplayName;
    FText                 Text;
    USoundBase*           Voice;
    FGameplayTagContainer EmotionTags;
    EMayDialogueAdvanceMode AdvanceMode;
    float                 AutoAdvanceDelay;
};
```

Die finale, zum UI gereichte Message-Struktur. Broadcast via `OnMessageReceived`.

### FMayDialogueChoiceEntry

```cpp
USTRUCT()
struct FMayDialogueChoiceEntry
{
    int32                          ChoiceIndex;
    FText                          Text;
    FGameplayTagContainer          Tags;
    bool                           bAvailable;
    FText                          UnavailableReason;
    FGuid                          TargetNodeGuid;
    EMayDialogueRequirementResult  RequirementResult;
};
```

Ein einzelner Choice-Eintrag nach Requirement-Evaluation und Filter.

### FMayDialogueBranchPoint

```cpp
USTRUCT()
struct FMayDialogueBranchPoint
{
    FText Description;
    TArray<UMayDialogueRequirement*> Requirements;
    FGuid TargetNodeGuid;
};
```

Ein Eintrag in `UDialogueNode_Branch::BranchPoints`. Reihenfolge bestimmt Auswertungs-Priorität.

### FMayDialogueContext

```cpp
USTRUCT()
struct FMayDialogueContext
{
    UMayDialogueInstance* Instance;
    AActor*               Instigator;
    AActor*               Target;
    UMayDialogueParticipant* Speaker;
    FGameplayTag          CurrentSpeakerTag;
};
```

Wird allen `ExecuteNode`-Calls gereicht. Aggregiert alles, was ein Node für Welt-Zugriffe braucht.

### FMayDialogueScopeEntry

```cpp
USTRUCT()
struct FMayDialogueScopeEntry
{
    UMayDialogueAsset* Asset;
    FGuid              ReturnNodeGuid;
};
```

Ein Frame auf dem Scope-Stack (Link / SubGraph). Siehe [Instance & Lifecycle → Links & Scope-Stack](../concepts/instance-lifecycle.md#links-scope-stack).

### FMayDialogueParticipants

```cpp
USTRUCT()
struct FMayDialogueParticipants
{
    TMap<FGameplayTag, TWeakObjectPtr<UMayDialogueParticipant>> ByTag;
};
```

Auflösungs-Container, den die Instance beim Start baut. Abfrage per Tag → Participant.

## Siehe auch

* [Instance & Lifecycle](../concepts/instance-lifecycle.md) – wie die State-Enums durchwandert werden.
* [GAS → Requirements](../gas/requirements.md) – wie `EMayDialogueRequirementResult` interpretiert wird.
* [Audio → Babel-System](../audio/babel-system.md) – `EMayDialogueBabelMode/SyncMode`.
* [Runtime → Read/Write-API](../runtime/read-write-api.md) – `EMayDialogueVariableType/Scope`.
