---
description: Alle Delegate-Typen mit Signatur, Feuer-Zeitpunkt und typischen Einsatzfällen.
---

# API: Delegates & Events

MayDialogue broadcastet auf zwei Ebenen: **Instance** (pro Gespräch) und **Subsystem** (global). Alle Delegates sind `DYNAMIC_MULTICAST_DELEGATE` — beliebig viele Listener, Blueprint-bindbar.

---

## Übersicht

| Delegate | Ebene | Feuer-Zeitpunkt |
|---|---|---|
| `OnDialogueStarted` | Instance | Direkt nach Instance-Erzeugung, vor erstem Node. |
| `OnDialogueEnded` | Instance | Bei Exit (Completed/Failed) oder Abort. |
| `OnNodeReached` | Instance | Nach erfolgreicher Node-Execution. |
| `OnMessageReceived` | Instance | Wenn SayLine eine Message aufbaut. |
| `OnChoicesPresented` | Instance | Wenn PlayerChoice-Node Optionen filtert und präsentiert. |
| `OnChoiceMade` | Instance | Nach `SelectChoice(Index)`, vor der Transition. |
| `OnVariableChanged` | Instance | Nach Variable-Mutation (Node oder externer Code). |
| `OnDialogueEventFired` | Instance | Bei FireEvent-Node. |
| `OnAnyDialogueStarted` | Subsystem | Jedes Mal wenn irgendein Dialog startet. |
| `OnAnyDialogueEnded` | Subsystem | Jedes Mal wenn irgendein Dialog endet. |

---

## Instance-Delegates

### OnDialogueStarted

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_FourParams(
    FOnMayDialogueStarted,
    UMayDialogueAsset*, Asset,
    AActor*,            Instigator,
    AActor*,            Target,
    float,              StartTime);

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnMayDialogueStarted OnDialogueStarted;
```

**Feuer-Zeitpunkt**: Nach Instance-Erzeugung, vor dem ersten Node.
**Einsatz**: Player-Bewegung einfrieren, Quest-Log-Eintrag anlegen, Musik-Switch.

---

### OnDialogueEnded

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_FiveParams(
    FOnMayDialogueEnded,
    UMayDialogueAsset*,     Asset,
    EMayDialogueExitStatus, ExitStatus,
    float,                  Duration,
    AActor*,                Instigator,
    AActor*,                Target);

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnMayDialogueEnded OnDialogueEnded;
```

**Feuer-Zeitpunkt**: `EndDialogue()` (Completed/Failed) oder `AbortDialogue()`.
**Einsatz**: Player-Bewegung freigeben, Quest-Abschluss prüfen, Follow-up-Dialog queuen.

---

### OnNodeReached

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(
    FOnMayDialogueNodeReached,
    FGuid,                 NodeGuid,
    UMayDialogueNode_Base*, Node);

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnMayDialogueNodeReached OnNodeReached;
```

**Feuer-Zeitpunkt**: Direkt nach `ExecuteNode(...)`.
**Einsatz**: Debug-HUD, Telemetrie, Breakpoint-System.

---

### OnMessageReceived

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnMayDialogueMessageReceived,
    const FMayDialogueMessage&, Message);

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnMayDialogueMessageReceived OnMessageReceived;
```

**Feuer-Zeitpunkt**: Innerhalb `SayLine::ExecuteNode`, nachdem `FMayDialogueMessage` gebaut wurde.
**Einsatz**: UI-Widget rendert Sprecher-Name, Text und Porträt.

---

### OnChoicesPresented

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnMayDialogueChoicesPresented,
    const TArray<FMayDialogueChoiceEntry>&, Choices);

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnMayDialogueChoicesPresented OnChoicesPresented;
```

**Feuer-Zeitpunkt**: `Instance::PresentChoices(...)`.
**Einsatz**: Choice-Buttons rendern, Timeout-Bar starten, Controller-Vibration.

---

### OnChoiceMade

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnMayDialogueChoiceMade,
    int32, ChoiceIndex);

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnMayDialogueChoiceMade OnChoiceMade;
```

**Feuer-Zeitpunkt**: `Instance::SelectChoice(Index)`, vor der Transition.
**Einsatz**: Analytics ("welche Choices wählen Spieler?"), Achievement-Trigger.

---

### OnVariableChanged

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_FourParams(
    FOnMayDialogueVariableChanged,
    FName,                     VariableName,
    EMayDialogueVariableScope, Scope,
    EMayDialogueVariableType,  Type,
    const FString&,            NewValueAsString);

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnMayDialogueVariableChanged OnVariableChanged;
```

**Feuer-Zeitpunkt**: Nach `SetVariable`-Node oder externem `SetDialogueVariable`-Call.
**Einsatz**: HUD-Update (Dispositions-Anzeige), Live-Quest-Status.

{% hint style="info" %}
Der Wert kommt als String. Mit `Type` weißt du wie du ihn zurück parsst (z.B. `FCString::Atoi` für Int).
{% endhint %}

---

### OnDialogueEventFired

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnMayDialogueEventFired,
    const FGameplayTag&, EventTag);

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnMayDialogueEventFired OnDialogueEventFired;
```

**Feuer-Zeitpunkt**: `FireEvent`-Action-Node im Dialog-Graphen.
**Einsatz**: Lose Kopplung zu externen Systemen — Lichteffekte, AI-Switch, Sound-Trigger.

---

## Subsystem-Delegates

### OnAnyDialogueStarted / OnAnyDialogueEnded

```cpp
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(
    FOnAnyDialogueEvent,
    UMayDialogueInstance*, Instance);

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnAnyDialogueEvent OnAnyDialogueStarted;

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnAnyDialogueEvent OnAnyDialogueEnded;
```

**Feuer-Zeitpunkt**: Subsystem-Level, parallel zur Instance-Variante.
**Einsatz**: Audio-Ducking, globaler Input-Mode-Switch, Analytics.

---

## Binding-Patterns

### Blueprint: Instance-Delegate binden

> 📸 **Bild-Platzhalter:** `delegate-bind-instance-bp.png` — BP-Graph: Start Dialogue → Bind Event to On Dialogue Ended.
> *Setup:* NPC-Blueprint. `Start Dialogue` (Return Value = Instance) → `Bind Event to On Dialogue Ended` (Instance-Pin = Return Value, Event-Pin = Custom Event `Handle Dialogue Ended`). Custom Event hat Params: `Asset`, `ExitStatus`, `Duration`, `Instigator`, `Target`. Darunter `Switch on EMayDialogueExitStatus`.

```text
[Start Dialogue] → Return Value (Instance)
    │
    ▼
[Bind Event to On Dialogue Ended]  ──► Custom Event: Handle Dialogue Ended
                                             (ExitStatus, Duration, ...)
```

### Blueprint: Subsystem-Delegate binden

```text
[Event BeginPlay]
    │
    ▼
[Get Dialogue Subsystem]
    │
    ▼
[Bind Event to On Any Dialogue Started]  ──► Custom Event: Handle Any Start
[Bind Event to On Any Dialogue Ended]    ──► Custom Event: Handle Any End
```

### C++: Instance-Delegate

```cpp
UMayDialogueInstance* Inst = Sub->StartDialogue(Asset, Player, NPC);
if (Inst)
{
    Inst->OnDialogueEnded.AddDynamic(this, &AMyActor::HandleEnd);
    Inst->OnVariableChanged.AddDynamic(this, &AMyActor::HandleVarChange);
}
```

### C++: Subsystem-Delegate + Cleanup

```cpp
// BeginPlay
Sub->OnAnyDialogueStarted.AddDynamic(this, &AMyActor::HandleStart);
Sub->OnAnyDialogueEnded.AddDynamic(this, &AMyActor::HandleEnd);

// EndPlay — Subsystem lebt bis Welt-Ende, daher sauber aufräumen
Sub->OnAnyDialogueStarted.RemoveDynamic(this, &AMyActor::HandleStart);
Sub->OnAnyDialogueEnded.RemoveDynamic(this, &AMyActor::HandleEnd);
```

---

## Delegate-Cheatsheet: Wann welches?

| Ziel | Bestes Delegate |
|---|---|
| Dialog-Overlay einblenden | `OnDialogueStarted` (Subsystem oder Instance) |
| Typewriter starten | `OnMessageReceived` |
| Choice-Buttons rendern | `OnChoicesPresented` |
| Wahl des Spielers loggen | `OnChoiceMade` |
| Quest-Schritt abschließen | `OnDialogueEnded` (ExitStatus == Completed) |
| HUD-Wert live aktualisieren | `OnVariableChanged` |
| Lichteffekt aus Dialog-Graphen triggern | `OnDialogueEventFired` |
| Player-Bewegung freigeben | `OnDialogueEnded` |
| Audio ducken für alle Dialoge | `OnAnyDialogueStarted` / `OnAnyDialogueEnded` am Subsystem |

## Siehe auch

- [Typen & Enums](types.md) — `EMayDialogueExitStatus`, `EMayDialogueVariableType`, `FMayDialogueMessage` etc.
- [Runtime → Bridge & Lifecycle-Events](../runtime/bridge-events.md) — geführter Walkthrough mit Code-Beispielen.
