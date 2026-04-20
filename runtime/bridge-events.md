# Bridge & Lifecycle-Events

Die **Bridge** ist das generische Extension-Interface, das externe Systeme (Quest, Achievements, Analytics, Debug-Tools) nutzen, um MayDialogue zu konsumieren **ohne harte Kopplung** auf das Subsystem.

## `IMayDialogueBridge`-Interface

```cpp
class IMayDialogueBridge
{
    virtual UMayDialogueInstance* StartDialogueFromBridge(UMayDialogueAsset*, AActor* Instigator, AActor* Target) = 0;
    virtual bool CanStartDialogue(UMayDialogueAsset*, AActor*, AActor*) const = 0;
    virtual bool IsDialogueActive() const = 0;
    virtual void AbortDialogue() = 0;
    virtual void SelectChoice(int32 Index) = 0;
    virtual void ForceAdvance() = 0;

    // Read
    virtual UMayDialogueAsset* GetActiveDialogueAsset() const = 0;
    virtual FGuid GetCurrentNodeGUID() const = 0;
    virtual TArray<AActor*> GetActiveParticipants() const = 0;
    virtual bool GetDialogueVariable(FName, EMayDialogueVariableType, FString& OutValueAsString) const = 0;
    virtual bool GetParticipantVariable(FGameplayTag, FName, EMayDialogueVariableType, FString& OutValueAsString) const = 0;
    virtual TArray<FMayDialogueChoiceEntry> GetPendingChoices() const = 0;

    // Write
    virtual bool SetDialogueVariable(FName, EMayDialogueVariableType, const FString& ValueAsString) = 0;
    virtual bool SetParticipantVariable(FGameplayTag, FName, EMayDialogueVariableType, const FString& ValueAsString) = 0;
};
```

**Das Subsystem implementiert dieses Interface vollständig.** Externe Systeme können also entweder direkt auf das Interface casten oder auf den Subsystem-Typ zeigen – beide Wege funktionieren.

### Warum ein Interface?

Damit ein externes System (z.B. MayFlowGraph oder ein eigenes Quest-System) **nicht** einen Hard-Dependency auf das `MayDialogue`-Modul braucht. Es importiert nur das `MayDialogueBridge.h`-Header und holt sich zur Laufzeit ein `IMayDialogueBridge*`-Objekt.

## Lifecycle-Delegates

Alle Delegates sitzen auf **`UMayDialogueInstance`** (Instanz-weite Events) und teilweise auf **`UMayDialogueSubsystem`** (globale Events).

### Auf der Instance

| Delegate | Wann | Parameter |
| --- | --- | --- |
| `OnDialogueStarted` | Bei StartDialogue | Asset, Instigator, Target, StartTime |
| `OnDialogueEnded` | Bei End/Abort | Asset, ExitStatus, Duration, Instigator, Target |
| `OnNodeReached` | Nach Node-Execution | NodeGuid, Node* |
| `OnMessageReceived` | Nach SayLine-Advance | `const FMayDialogueMessage&` |
| `OnChoicesPresented` | Wenn PlayerChoice aktiv wird | Array |
| `OnChoiceMade` | Nach SelectChoice | int32 ChoiceIndex |
| `OnVariableChanged` | Variable-Mutation | Name, Scope, Typ, NewValueAsString |
| `OnDialogueEventFired` | Bei FireEvent-Node | EventTag |

### Auf dem Subsystem

| Delegate | Wann | Parameter |
| --- | --- | --- |
| `OnAnyDialogueStarted` | Jedes Mal wenn irgendein Dialog startet | Instance |
| `OnAnyDialogueEnded` | Jedes Mal wenn irgendein Dialog endet | Instance |

Subsystem-Delegates sind für **globale Listener** (z.B. Audio-Ducking: *„immer wenn ein Dialog läuft, ducke die Background-Music"*).

Instance-Delegates sind für **spezifische Dialog-Listener** (z.B. Quest-System: *„wenn DIESER Dialog endet, prüfe den ExitStatus"*).

## Beispiel: Quest-System hört auf Dialog-Ende

```cpp
void AQuestDirector::OnWorldBeginPlay(UWorld& World)
{
    Super::OnWorldBeginPlay(World);
    if (auto* Sub = UMayDialogueSubsystem::Get(&World))
    {
        Sub->OnAnyDialogueStarted.AddDynamic(this, &AQuestDirector::HandleDialogStart);
        Sub->OnAnyDialogueEnded.AddDynamic(this, &AQuestDirector::HandleDialogEnd);
    }
}

void AQuestDirector::HandleDialogEnd(UMayDialogueInstance* Instance)
{
    if (!Instance) return;

    // Welcher Dialog war das, welchen Status hatte er?
    UMayDialogueAsset* Asset = Instance->GetDialogueAsset();
    EMayDialogueExitStatus Status = Instance->GetExitStatus();

    if (Asset == QuestDialogRef && Status == EMayDialogueExitStatus::Completed)
    {
        CompleteQuestStep();
    }
}
```

## Beispiel: Analytics-Logger

```cpp
void UAnalyticsLogger::Start(UMayDialogueInstance* Instance)
{
    Instance->OnChoiceMade.AddDynamic(this, &ThisClass::HandleChoice);
}

void UAnalyticsLogger::HandleChoice(int32 ChoiceIndex)
{
    // Hole die Tags der gewählten Choice
    // und schicke sie ans Analytics-Backend
}
```

## Anmerkungen

* **Alle Delegates sind Multicast** – beliebig viele Listener parallel.
* **Instance-Delegates werden mit der Instance zerstört.** Wenn du einen Listener brauchst, der mehrere Dialoge beobachtet, nutze die Subsystem-Delegates und hole dir die Instance aus deren Parametern.
* Die Bridge-Write-API ist **Live-Modifikation** – SetDialogueVariable während eines aktiven Dialogs triggert sofort einen `OnVariableChanged`-Event und kann Choice-Availabilities ändern.
