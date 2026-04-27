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
| `Aborted` | Extern unterbrochen (neuer Dialog, Level-Travel, `StopAllDialogues`). |

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

Wann Babel-Voice-Synthese einspringt.

| Wert | Bedeutung |
|---|---|
| `Off` | Nie. |
| `FallbackOnly` | Nur wenn kein Voice-Asset vorhanden. |
| `Always` | Immer — auch wenn Voice-Asset gesetzt ist (Preview-Modus). |

---

### EMayDialogueBabelSyncMode

Wie Babel mit dem Typewriter synchronisiert.

| Wert | Bedeutung |
|---|---|
| `PerCharacter` | Ein Laut pro Typewriter-Zeichen. |
| `PerWord` | Ein Babel-Block pro Wort. |
| `Continuous` | Kontinuierlicher Ton, unabhängig vom Typewriter-Takt. |

---

## Structs

### FMayDialogueMessage

Die fertige Message-Struktur, die via `OnMessageReceived` ans UI geliefert wird.

```cpp
USTRUCT()
struct FMayDialogueMessage
{
    FGameplayTag          SpeakerTag;       // Welcher Sprecher
    FText                 DisplayName;      // Anzeigename
    FText                 Text;             // Dialog-Text
    USoundBase*           Voice;            // Voice-Asset (nullptr wenn keins)
    FGameplayTagContainer EmotionTags;      // Emotion-Kontext
    EMayDialogueAdvanceMode AdvanceMode;    // Wie wird nach dieser Line advanced
    float                 AutoAdvanceDelay; // Delay bei AdvanceMode::Timer
};
```

---

### FMayDialogueChoiceEntry

Ein einzelner Choice-Eintrag nach Requirement-Evaluation und Filter.

```cpp
USTRUCT()
struct FMayDialogueChoiceEntry
{
    int32                         ChoiceIndex;        // 0-basierter Index für SelectChoice()
    FText                         Text;               // Anzeigetext der Choice
    FGameplayTagContainer         Tags;               // Tags der Choice (für Analytics, Styling)
    bool                          bAvailable;         // true = wählbar
    FText                         UnavailableReason;  // Grund wenn nicht wählbar (für Greyed-UI)
    FGuid                         TargetNodeGuid;     // Ziel-Node bei Auswahl
    EMayDialogueRequirementResult RequirementResult;  // Passed / FailedButVisible / FailedAndHidden
};
```

---

### FMayDialogueSpeaker

Eintrag im Speakers-Panel des Dialog-Asset-Editors.

```cpp
USTRUCT()
struct FMayDialogueSpeaker
{
    FGameplayTag SpeakerTag;                               // Eindeutiger Sprecher-Identifier
    FText        DisplayName;                              // Anzeigename im UI
    TSoftObjectPtr<UTexture2D> Portrait;                   // Porträt-Textur
    FGameplayTagContainer DefaultEmotionTags;              // Standard-Emotions
    TSoftObjectPtr<USoundClass> VoiceSoundClassOverride;   // Sound-Class-Überschreibung
    TSoftObjectPtr<UMayDialogueBabelProfile> BabelProfileOverride; // Babel-Profil-Überschreibung
};
```

---

### FMayDialogueContext

Wird allen `ExecuteNode`-Calls mitgegeben. Aggregiert alles was ein Node für Welt-Zugriffe braucht.

```cpp
USTRUCT()
struct FMayDialogueContext
{
    UMayDialogueInstance*    Instance;          // Die laufende Instance
    AActor*                  Instigator;        // Wer den Dialog gestartet hat
    AActor*                  Target;            // Mit wem gesprochen wird
    UMayDialogueParticipant* Speaker;           // Aktiver Sprecher (Participant-Komponente)
    FGameplayTag             CurrentSpeakerTag; // Tag des aktiven Sprechers
};
```

---

### FMayDialogueParticipants

Container der Instance für den Participant-Lookup per Tag.

```cpp
USTRUCT()
struct FMayDialogueParticipants
{
    TMap<FGameplayTag, TWeakObjectPtr<UMayDialogueParticipant>> ByTag;
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

Ein einzelner Zweig in einem Branch-Node.

```cpp
USTRUCT()
struct FMayDialogueBranchPoint
{
    FText Description;                           // Redaktionelle Beschriftung
    TArray<UMayDialogueRequirement*> Requirements; // Bedingungen (alle müssen Passed liefern)
    FGuid TargetNodeGuid;                        // Ziel-Node wenn Bedingungen erfüllt
};
```

Die Reihenfolge der `BranchPoints` bestimmt die Auswertungs-Priorität (erster Treffer gewinnt).

---

## Siehe auch

- [Runtime → Read/Write-API](../runtime/read-write-api.md) — `EMayDialogueVariableType/Scope` in der Praxis.
- [API: Delegates](api-delegates.md) — wie `FMayDialogueMessage` und `FMayDialogueChoiceEntry` in Delegate-Parametern vorkommen.
- [Konzepte → Instance & Lifecycle](../concepts/instance-lifecycle.md) — wie die Status-Enums durchlaufen werden.
- [GAS → Requirements](../gas/requirements.md) — wie `EMayDialogueRequirementResult` ausgewertet wird.
