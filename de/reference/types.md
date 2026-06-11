---
description: Alle Enums und wichtigen Structs des Plugins — Werte, Felder, Ein-Satz-Erklärungen.
---

# Typen & Enums (Referenz)

Alle öffentlichen Enums und Structs in einer Tabelle. Für tiefergehende Erklärungen verlinkt jede Gruppe auf das zugehörige Kapitel.

---

## Enums

### EMayDialogueStatus

Lifecycle-Zustand einer laufenden Instance. Direkt lesbar als `Instance->Status`.

| Wert | Bedeutung |
|---|---|
| `Inactive` | Initialer Zustand, noch nicht gestartet. |
| `Active` | Läuft, kein Warten. |
| `WaitingForAdvance` | Wartet auf Spieler-Input oder Timer (SayLine). |
| `WaitingForChoice` | PlayerChoice wurde präsentiert, Spieler muss wählen. |
| `Completed` | Exit-Node mit Completed-Status erreicht. |
| `Aborted` | Extern unterbrochen. |

---

### EMayDialogueExitStatus

Ergebnis eines Dialog-Endes. Parameter in `OnDialogueEnded`.

| Wert | Bedeutung |
|---|---|
| `Completed` | Sauber beendet — Exit-Node reached. |
| `Failed` | Exit-Node mit Failed-Status oder Requirement-Abort. |
| `Aborted` | Extern unterbrochen (neuer Dialog, Level-Travel, `AbortAllDialogues`). |

---

### EMayDialogueAdvanceMode

Steuert wann eine SayLine zum nächsten Node weitergeht.

| Wert | Bedeutung |
|---|---|
| `Manual` | Spieler-Input (Klick / Tastendruck). |
| `Timer` | Nach `AutoAdvanceDelay`-Sekunden. |
| `AfterVoice` | Nach Ende der Voice-Wiedergabe (Fallback: Typewriter-Ende). |
| `AfterAnimation` | Nach Ende der aktuellen Montage am Speaker. |
| `Immediate` | Kein Warten, sofort weiter (für reine Logik-Nodes). |

---

### EMayDialogueVariableType

Typen die in Dialog- und Participant-Variablen erlaubt sind.

| Wert | C++-Typ | String-Format |
|---|---|---|
| `Bool` | `bool` | `"true"` / `"false"` |
| `Int` | `int32` | `"42"` |
| `Float` | `float` | `"3.14"` |
| `String` | `FString` | beliebig |
| `Tag` | `FGameplayTag` | `"Dialogue.Mood.Friendly"` |

---

### EMayDialogueVariableScope

Wo eine Variable gespeichert wird.

| Wert | Bedeutung | Lebensdauer |
|---|---|---|
| `Dialogue` | Nur für die Lebensdauer der Instance. | Endet mit dem Dialog. |
| `Participant` | Im `PersistentMemory` der Participant-Komponente. | Überdauert Gespräche (SaveGame-Feld). |

---

### EMayDialogueRequirementResult

Ergebnis einer Requirement-Evaluation. Bestimmt wie der Choice-Filter reagiert.

| Wert | Bedeutung |
|---|---|
| `Passed` | Requirement erfüllt — Choice verfügbar. |
| `FailedButVisible` | Nicht erfüllt, Choice wird greyed-out angezeigt. |
| `FailedAndHidden` | Nicht erfüllt, Choice wird komplett ausgeblendet. |

---

### EMayDialogueNodeFailBehavior

Was passiert wenn ein Node seine Requirements nicht erfüllt.

| Wert | Bedeutung |
|---|---|
| `Skip` | Node überspringen, am Output-Pin weiter. |
| `Abort` | Dialog mit `EMayDialogueExitStatus::Failed` beenden. |

---

### EMayDialogueAudioMode

Spatial-Mode für Voice-Wiedergabe.

| Wert | Bedeutung |
|---|---|
| `Default` | Projekt-Default (`bForce2D` aus Project Settings). |
| `Spatial3D` | 3D-positioniert am Speaker-Actor. |
| `Force2D` | Immer 2D, ignoriert Project-Default. |

---

### EMayDialogueBabelMode

Steuert den Babel-Synthese-Algorithmus pro Profil.

| Wert | Bedeutung |
|---|---|
| `BlipPerCharacter` | Ein kurzer Blip-Sound pro enthülltem Zeichen ("bip bip bip"). Klassischer VN-Stil. |
| `PhonemeBase` | Sinusoidale Töne basierend auf Vokal-/Konsonant-Typ. Für abstrakte oder Kreatur-Stimmen. |

---

### EMayDialogueBabelSyncMode

Wie Babel mit dem Typewriter synchronisiert.

| Wert | Bedeutung |
|---|---|
| `TypewriterSync` | Reagiert auf jeden enthüllten Buchstaben des Typewriters. Präzise Zeichen-genaue Synchronisation. |
| `Continuous` | Interner Timer, unabhängig vom Typewriter-Takt. Für Kreatur-Groll oder Nicht-Typewriter-Abläufe. |

---

### EMayDialogueBabelEngine

Wählt das Synthese-Back-End von `UMayDialogueBabelSynth`. Wird im `UMayDialogueBabelProfile`-Asset konfiguriert.

| Wert | Bedeutung |
|---|---|
| `Granular` | **Standard.** Spielt voraufgenommene Blip-Samples aus den Profil-Pools mit zufälligem Pitch/Volume-Jitter. Fears-to-Fathom / Animal-Crossing-Qualität. |
| `Biquad` | Legacy-prozedurale Formant-Synthese (RBJ-Bandpass-Filter-Kette). Opt-in für Projekte, die den originalen DSP-Pfad nutzten. Anzeigename: *Biquad Synthesizer (Legacy)*. |

---

## Structs

### FMayDialogueMessage

Die fertige Message-Struktur, die via `OnMessageReceived` ans UI geliefert wird.

```cpp
USTRUCT(BlueprintType)
struct FMayDialogueMessage
{
    FGameplayTag               SpeakerTag;         // Welcher Sprecher
    FText                      SpeakerDisplayName; // Anzeigename im UI
    FText                      Text;               // Dialog-Text
    TSoftObjectPtr<UTexture2D> SpeakerPortrait;    // Porträt (vom Widget async geladen)
    TSoftObjectPtr<USoundBase> Voice;              // Voice-Asset (Soft-Ref; IsNull() wenn keins)
    FGameplayTagContainer      EmotionTags;        // Emotion-Kontext
    EMayDialogueAdvanceMode    AdvanceMode;        // Wie wird nach dieser Line advanced
    float                      AutoAdvanceDelay;   // Delay bei AdvanceMode::Timer
};
```

`Voice` und `SpeakerPortrait` sind **Soft-Referenzen** (`TSoftObjectPtr`), sodass sie ein Netz-RPC überleben, auch wenn sie auf dem empfangenden Client noch nicht geladen sind — siehe [Multiplayer](../runtime/multiplayer.md) für das Streaming-Detail.

---

### FMayDialogueChoiceEntry

Ein einzelner Choice-Eintrag nach Requirement-Evaluation und Filter.

```cpp
USTRUCT(BlueprintType)
struct FMayDialogueChoiceEntry
{
    FText                         ChoiceText;         // Anzeigetext der Choice
    FGameplayTagContainer         ChoiceTags;         // Tags der Choice (für Analytics, Styling)
    EMayDialogueRequirementResult AvailabilityStatus; // Passed / FailedButVisible / FailedAndHidden
    FText                         UnavailableReason;  // Grund wenn nicht wählbar (für Greyed-UI)
    int32                         ChoiceIndex;        // Authored-Index in der vollen Choice-Liste
    FGuid                         TargetNodeGuid;     // Ziel-Node bei Auswahl
    float                         Weight;             // Gewicht für Weighted-Random-Default bei Timeout
};
```

Die Helfer `IsAvailable()` (Status == `Passed`) und `IsVisible()` (Status != `FailedAndHidden`) lesen `AvailabilityStatus`.

{% hint style="warning" %}
`ChoiceIndex` ist der **authored** Index in der vollen Choice-Liste des Nodes. `SelectChoice` / `ServerSelectChoice` erwarten die **Position** des Eintrags in der *präsentierten sichtbaren* Liste, die abweicht, wenn eine versteckte Choice davor liegt. Übergib die Position des Eintrags im empfangenen Array, nicht `ChoiceIndex` — außer du weißt, dass keine Choice versteckt ist.
{% endhint %}

---

### FMayDialogueSpeaker

Eintrag im Speakers-Panel des Dialog-Asset-Editors.

```cpp
USTRUCT()
struct FMayDialogueSpeaker
{
    FGameplayTag SpeakerTag;                                  // Eindeutiger Sprecher-Identifier
    FText        DisplayName;                                 // Anzeigename im UI
    TSoftObjectPtr<UTexture2D> Portrait;                      // Porträt-Textur
    FLinearColor NodeColor;                                   // Editor-only Graph-Tint
    EMayDialogueAudioMode AudioModeOverride;                  // 2D/3D-Override pro Sprecher
    TSoftObjectPtr<USoundClass> SoundClassOverride;           // Sound-Class-Überschreibung
    TSoftObjectPtr<USoundAttenuation> AttenuationOverride;    // 3D-Attenuation-Überschreibung
    float VolumeMultiplier;                                   // Volume-Skalar pro Sprecher (Default 1.0)
    float PitchMultiplier;                                    // Pitch-Skalar pro Sprecher (Default 1.0)
    TSoftObjectPtr<UMayDialogueBabelProfile> BabelProfile;    // Babel-Profil pro Sprecher
};
```

---

### FMayDialogueContext

Wird allen `ExecuteNode`-Calls mitgegeben. Aggregiert alles was ein Node für Welt-Zugriffe braucht.

```cpp
USTRUCT(BlueprintType)
struct FMayDialogueContext
{
    TWeakObjectPtr<UMayDialogueInstance> DialogueInstance;       // Die laufende Instance (Weak-Ref)
    TObjectPtr<AActor>                   Instigator;             // Wer den Dialog gestartet hat
    TObjectPtr<AActor>                   Target;                 // Mit wem gesprochen wird
    FGuid                                CurrentNodeGuid;        // Aktuell betretener Node (mid-traversal)
    bool                                 bOwningNodeReachCounted; // True wenn der aktuelle Reach schon gezählt ist

    UObject* ResolveInstigatorASC() const; // ASC des Instigators, oder nullptr
    UWorld*  GetWorld() const;
};
```

---

### FMayDialogueParticipants

Container der Instance für den Participant-Lookup per Tag.

```cpp
USTRUCT()
struct FMayDialogueParticipants
{
    TMap<FGameplayTag, TWeakObjectPtr<UMayDialogueParticipant>> ParticipantMap;
    // Nutze die Helfer, nicht die Map direkt:
    UMayDialogueParticipant*          FindParticipant(const FGameplayTag& Tag) const;
    TArray<UMayDialogueParticipant*>  GetAllParticipants() const;
};
```

---

### FMayDialogueScopeEntry

Ein Frame auf dem internen Scope-Stack (für Link- und SubGraph-Nodes).

```cpp
USTRUCT()
struct FMayDialogueScopeEntry
{
    UMayDialogueAsset* Asset;          // Asset des Sub-Dialogs
    FGuid              ReturnNodeGuid; // Wohin nach dem Sub-Dialog zurückgekehrt wird
};
```

---

### FMayDialogueBranchPoint

Eine Gruppierungs-Struktur, die intern ein Set von Choice-Einträgen trägt.

```cpp
USTRUCT(BlueprintType)
struct FMayDialogueBranchPoint
{
    TArray<FMayDialogueChoiceEntry> Choices;
};
```

{% hint style="info" %}
Das Branch-*Routing* selbst liegt auf dem `UMayDialogueNode_Branch`-Node: Er wertet eine einzelne `Condition`-Requirement aus und nimmt den **True**-Output, wenn sie passt, den **False**-Output, wenn sie fehlschlägt, und einen optionalen **Default**-Output, wenn `bHasFallback` gesetzt ist — nicht auf dieser Struct. Siehe [Branch-Node](../nodes/core/branch.md).
{% endhint %}

---

## Siehe auch

- [Runtime → Read/Write-API](../runtime/read-write-api.md) — `EMayDialogueVariableType/Scope` in der Praxis.
- [API: Delegates](api-delegates.md) — wie `FMayDialogueMessage` und `FMayDialogueChoiceEntry` in Delegate-Parametern vorkommen.
- [Konzepte → Instance & Lifecycle](../concepts/instance-lifecycle.md) — wie die Status-Enums durchlaufen werden.
- [GAS → Requirements](../gas/requirements.md) — wie `EMayDialogueRequirementResult` ausgewertet wird.
