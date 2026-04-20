# Delegates & Events (Referenz)

MayDialogue broadcastet an zwei Ebenen: **pro Instance** (Lifecycle-Events eines einzelnen Gesprächs) und **pro Subsystem** (globale „irgendein Dialog"-Events). Alle Delegates sind `DYNAMIC_MULTICAST_DELEGATE*` und damit Blueprint-bindbar.

## Übersicht

| Delegate | Ebene | Feuer-Punkt |
| --- | --- | --- |
| `OnDialogueStarted` | Instance | Direkt nach Instance-Erzeugung, vor Entry-Execution. |
| `OnDialogueEnded` | Instance | Bei Exit (Completed/Failed) oder Abort. |
| `OnNodeReached` | Instance | Nach erfolgreicher Node-Execution. |
| `OnMessageReceived` | Instance | Wenn eine SayLine-Message feststeht. |
| `OnChoicesPresented` | Instance | Wenn PlayerChoice gebaut + gefiltert ist. |
| `OnChoiceMade` | Instance | Nach `SelectChoice(Index)`. |
| `OnVariableChanged` | Instance | Nach Variable-Mutation via SetVariable. |
| `OnDialogueEventFired` | Instance | Bei FireEvent-Node. |
| `OnAnyDialogueStarted` | Subsystem | Pro startender Instance. |
| `OnAnyDialogueEnded` | Subsystem | Pro endender Instance. |

## Signaturen

### OnDialogueStarted (Instance)

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_FourParams(
    FOnDialogueStarted,
    UMayDialogueAsset*, Asset,
    AActor*,            Instigator,
    AActor*,            Target,
    float,              StartTime);
```

**Feuer-Punkt**: `UMayDialogueSubsystem::StartDialogue`, Schritt 4.
**Einsatz**: Quest-Log-Eintrag anlegen, Player-Bewegung einfrieren, Musik-Switch.

### OnDialogueEnded (Instance)

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_FiveParams(
    FOnDialogueEnded,
    UMayDialogueAsset*,       Asset,
    EMayDialogueExitStatus,   ExitStatus,
    float,                    Duration,
    AActor*,                  Instigator,
    AActor*,                  Target);
```

**Feuer-Punkt**: `EndDialogue(Completed/Failed)` oder `AbortDialogue`.
**Einsatz**: Player-Bewegung wieder freigeben, Quest-Abschluss prüfen, Follow-up-Dialog queuen.

### OnNodeReached (Instance)

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnNodeReached,
    FGuid,                    NodeGuid,
    UMayDialogueNodeBase*,    Node);
```

**Feuer-Punkt**: Direkt nach `Node->ExecuteNode(...)`.
**Einsatz**: Debug-HUD, Breakpoint-System, Analytics.

### OnMessageReceived (Instance)

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnMessageReceived,
    const FMayDialogueMessage&, Message);
```

**Feuer-Punkt**: Innerhalb von `SayLine::ExecuteNode`, nachdem `FMayDialogueMessage` gebaut wurde.
**Einsatz**: Das UI hängt sich hier rein und rendert den Text.

### OnChoicesPresented (Instance)

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnChoicesPresented,
    const TArray<FMayDialogueChoiceEntry>&, Choices);
```

**Feuer-Punkt**: `Instance::PresentChoices(...)`.
**Einsatz**: Choice-List-Widget rendert, Timeout-Bar startet, Controller-Vibration.

### OnChoiceMade (Instance)

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnChoiceMade,
    int32, ChoiceIndex);
```

**Feuer-Punkt**: `Instance::SelectChoice(Index)`, vor der Transition.
**Einsatz**: Analytics (welche Choice wählen Spieler?), Achievement-Trigger.

### OnVariableChanged (Instance)

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_FourParams(
    FOnVariableChanged,
    FName,                          VariableName,
    EMayDialogueVariableScope,      Scope,
    EMayDialogueVariableType,       Type,
    const FString&,                 NewValueAsString);
```

**Feuer-Punkt**: Nach SetVariable (Action-Node oder SideEffect).
**Einsatz**: HUD-Update (z.B. Disposition-Anzeige), Live-Quest-Status.

{% hint style="info" %}
Der Value kommt **als String**. Für typspezifische Werte parse zurück je nach `Type` oder nutze die `UMayDialogueInstance::GetDialogueVariableTyped<T>`-Helpers.
{% endhint %}

### OnDialogueEventFired (Instance)

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnDialogueEventFired,
    FGameplayTag, EventTag);
```

**Feuer-Punkt**: FireEvent-Action-Node.
**Einsatz**: Lose Kopplung zu weltfremden Systemen (Audio, Lichteffekt, AI-Switch).

### OnAnyDialogueStarted / OnAnyDialogueEnded (Subsystem)

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnAnyDialogueEvent,
    UMayDialogueInstance*, Instance);
```

**Feuer-Punkt**: Subsystem-Level, parallel zur Instance-Variante.
**Einsatz**: Globale Game-Systems (Quest-Log, Music-Mixer, Save-Trigger).

## Binding-Patterns

### C++ - Einzelne Instance

```cpp
UMayDialogueInstance* Inst = Sub->StartDialogue(Asset, Player, NPC);
if (Inst)
{
    Inst->OnDialogueEnded.AddDynamic(this, &AMyPawn::HandleDialogueEnded);
}
```

### C++ - Global (Subsystem)

```cpp
UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(GetWorld());
Sub->OnAnyDialogueStarted.AddDynamic(this, &AQuestLog::HandleAnyStart);
Sub->OnAnyDialogueEnded.AddDynamic(this, &AQuestLog::HandleAnyEnd);
```

### Blueprint

```
[Get Dialogue Subsystem]
   │
   ▼
[Bind Event to On Any Dialogue Started]
   └─► Custom Event: Handle Dialogue Started
```

Für Instance-Delegates: Rückgabewert von `Start Dialogue` nehmen und `Bind Event to On Dialogue Ended` am Instance-Pin setzen.

## Unbind nicht vergessen

Wenn der bindende Actor vor dem Dialog-Ende zerstört wird, broadcastet UE trotzdem weiter. Das Multicast-Delegate prüft `IsValid(Target)` – insofern meistens ungefährlich, aber sauber ist:

```cpp
BeginPlay:   AddDynamic
EndPlay:     RemoveDynamic
```

Für Subsystem-Delegates gilt das stärker – die leben bis Welt-Ende.

## Reihenfolge bei mehreren Listenern

`DYNAMIC_MULTICAST` garantiert **keine bestimmte Reihenfolge**. Wenn dein System darauf angewiesen ist, dass z.B. das UI *nach* dem Quest-Log updatet, nutze Prioritäts-Queues in deinem Game-Code oder chain explizit.

## Häufige Szenarien – Delegate-Cheatsheet

| Szenario | Bestes Delegate |
| --- | --- |
| Dialog-Overlay einblenden | `OnDialogueStarted` (Subsystem oder Instance) |
| Typewriter starten | `OnMessageReceived` |
| Choice-Buttons rendern | `OnChoicesPresented` |
| Achievement „10 Gute Choices" | `OnChoiceMade` |
| Quest-Log-Eintrag aktualisieren | `OnVariableChanged` oder `OnDialogueEventFired` |
| Player-Movement re-enablen | `OnDialogueEnded` |
| Analytics „Dialog abgebrochen" | `OnDialogueEnded` + Filter auf `ExitStatus == Aborted` |

## Siehe auch

* [`UMayDialogueSubsystem`](api-subsystem.md)
* [Typen & Enums](types.md) – `EMayDialogueExitStatus`, `EMayDialogueVariableType` etc.
* [Runtime → Bridge & Lifecycle-Events](../runtime/bridge-events.md)
