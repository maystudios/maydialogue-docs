# Read/Write-API

Externe Systeme können Dialog-State **lesen und schreiben**. Nützlich für Cheat-Menüs, Tutorial-Skripte, Analytics, Debug-Tools.

## Read-API

| Methode | Rückgabe |
| --- | --- |
| `GetActiveDialogueAsset()` | Aktives Asset oder nullptr |
| `GetCurrentNodeGUID()` | GUID des ausführenden Nodes |
| `GetActiveParticipants()` | Array aller teilnehmenden Actors |
| `GetDialogueVariable(Name, Type, Out)` | Dialogue-Scope-Variable als String |
| `GetParticipantVariable(ParticipantTag, Name, Type, Out)` | Participant-Scope-Variable |
| `GetPendingChoices()` | Aktueller `FMayDialogueChoiceEntry`-Array oder leer |

Alle Methoden sind am `IMayDialogueBridge`-Interface definiert und vom Subsystem implementiert.

### C++-Beispiel

```cpp
UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(World);
if (!Sub->IsDialogueActive()) return;

FString AngerValue;
bool bFound = Sub->GetDialogueVariable(
    "AngerLevel",
    EMayDialogueVariableType::Int,
    AngerValue
);

if (bFound)
{
    int32 Anger = FCString::Atoi(*AngerValue);
    UE_LOG(LogMyGame, Log, TEXT("Current anger: %d"), Anger);
}
```

## Write-API

| Methode | Wirkung |
| --- | --- |
| `SetDialogueVariable(Name, Type, Value)` | Dialog-Scope-Variable setzen |
| `SetParticipantVariable(ParticipantTag, Name, Type, Value)` | Participant-Scope-Variable setzen |
| `SelectChoice(Index)` | Aktive Choice programmatisch auswählen |
| `ForceAdvance()` | Aktuellen Advance-Wait überspringen |
| `AbortDialogue()` | Dialog abbrechen |

### C++-Beispiel: Cheat „alle Requirements Passed"

```cpp
void UCheatManager::UnblockAllChoices()
{
    auto* Sub = UMayDialogueSubsystem::Get(GetWorld());
    if (!Sub->IsDialogueActive()) return;

    // Setze beliebige Projekt-Tags / Variablen, die Requirements prüfen
    Sub->SetDialogueVariable("CheatMode", EMayDialogueVariableType::Bool, "true");
}
```

### C++-Beispiel: Auto-Play für Tests

```cpp
void UAutoPlayer::Tick(float DeltaSeconds)
{
    auto* Sub = UMayDialogueSubsystem::Get(GetWorld());
    if (!Sub->IsDialogueActive()) return;

    TArray<FMayDialogueChoiceEntry> Choices = Sub->GetPendingChoices();
    if (Choices.Num() > 0)
    {
        Sub->SelectChoice(0);  // wähle immer die erste Option
        return;
    }

    Sub->ForceAdvance();
}
```

## Blueprint-Zugriff

Alle Bridge-Methoden sind `BlueprintCallable`. Du findest sie unter dem Subsystem-Node oder als Library-Wrapper.

## Typen-String-Serialisierung

Die Variable-API arbeitet mit **String-Repräsentationen**:

| Typ | String-Format |
| --- | --- |
| Bool | `"true"` / `"false"` |
| Int | `"42"` |
| Float | `"3.14"` |
| String | beliebig |
| Tag | voller Tag-Pfad, z.B. `"Dialogue.Mood.Friendly"` |

Parsing / Conversion passiert im Subsystem; du musst nur den richtigen Typ angeben.

## Einsatzfälle

| Szenario | Lösung |
| --- | --- |
| Cheat-Menü setzt Dialog-Variable | `SetDialogueVariable` |
| Tutorial schaltet durch einen Dialog durch | `ForceAdvance` + `SelectChoice` |
| Analytics loggt Dialog-State | `GetActiveDialogueAsset`, `GetCurrentNodeGUID`, `GetDialogueVariable` |
| Savegame serialisiert Participant-State | über `UMayDialogueSaveHelper` (bequemer) |
| Debug-UI zeigt alle Dialog-Variablen | `GetDialogueVariable` in Schleife |

## Anmerkungen

* `SelectChoice` triggert die normale Choice-SideEffect-Kaskade – keine Short-Cuts.
* `ForceAdvance` respektiert den aktuellen Advance-Mode nicht, überspringt Timer / AfterVoice und geht sofort zum Output.
* Die **string-basierte API** ist absichtlich – sie macht Cross-System-Kopplung trivial, ohne dass externe Systeme wissen müssen, welche konkreten Typen die Variable hat.
