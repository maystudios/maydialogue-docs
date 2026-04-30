---
description: Lifecycle-Delegates von MayDialogue — wann sie feuern und wie du dich einbindest.
---

# Bridge & Lifecycle-Events

MayDialogue broadcastet an zwei Ebenen: **Instance-Delegates** für Events eines einzelnen Gesprächs und **Subsystem-Delegates** für globale "irgendein Dialog läuft"-Events.

## Überblick: Wann feuert was?

| Delegate | Ebene | Feuert wenn... |
|---|---|---|
| `OnDialogueStarted` | Instance | Die Instance startet, direkt vor dem ersten Node. |
| `OnDialogueEnded` | Instance | Dialog endet — egal ob Completed, Failed oder Aborted. |
| `OnNodeReached` | Instance | Ein Node erfolgreich ausgeführt wurde. |
| `OnMessageReceived` | Instance | Eine SayLine eine Message an die UI liefert. |
| `OnChoicesPresented` | Instance | Ein PlayerChoice-Node Optionen aufbaut und filtert. |
| `OnChoiceMade` | Instance | Der Spieler (oder Code) eine Choice auswählt. |
| `OnVariableChanged` | Instance | Eine Dialog-Variable oder Participant-Variable sich ändert. |
| `OnDialogueEventFired` | Instance | Ein FireEvent-Node einen GameplayTag sendet. |
| `OnAnyDialogueStarted` | Subsystem | Jedes Mal wenn irgendein Dialog startet. |
| `OnAnyDialogueAborted` | Subsystem | Wenn irgendein Dialog abgebrochen wird — feuert **vor** `OnAnyDialogueEnded`. |
| `OnAnyDialogueEnded` | Subsystem | Jedes Mal wenn irgendein Dialog endet (Completed oder Aborted). |

---

## Subsystem-Delegates (global)

Nutze diese wenn du auf **alle Dialoge** reagieren willst — unabhängig davon welcher Asset oder NPC beteiligt ist.

### OnAnyDialogueAborted

Feuert wenn eine Instance mit `ExitStatus == Aborted` endet — **vor** `OnAnyDialogueEnded`.

**Signatur:** `Asset, ExitStatus, Duration (float), Instigator (AActor*), Target (AActor*)`

**Typische Use-Cases:**
- Interrupted-Dialog als "unfinished" in Quest-Log markieren
- Analytics: abgebrochenen Gesprächs-Versuch loggen
- Aufräumen von temporärem State ohne false "Completed"-Auslösung

```cpp
Sub->OnAnyDialogueAborted.AddDynamic(this, &AMyActor::HandleAborted);

void AMyActor::HandleAborted(
    UMayDialogueAsset* Asset,
    EMayDialogueExitStatus ExitStatus,
    float Duration,
    AActor* Instigator,
    AActor* Target)
{
    UAnalyticsLibrary::LogEvent("dialogue_aborted", Asset->GetName());
}
```

{% hint style="info" %}
`OnAnyDialogueEnded` wird danach immer gefeuert — egal ob Completed oder Aborted. Wenn du beide bindest, erhältst du erst `OnAnyDialogueAborted`, dann `OnAnyDialogueEnded`.
{% endhint %}

---

### OnAnyDialogueStarted / OnAnyDialogueEnded

**Typische Use-Cases:**
- Audio-Ducking: Musik leiser schalten wenn ein Dialog läuft
- Input-Mode global auf GameAndUI umschalten
- Analytics: jeden Dialogstart loggen

#### Blueprint

> 📸 **Bild-Platzhalter:** `subsystem-delegate-bind-bp.png` — Subsystem-Delegate-Binding im BeginPlay.
> *Setup:* GameMode- oder GameInstance-Blueprint. `Event BeginPlay` → `Get MayDialogue Subsystem` → Verbindung zu `Bind Event to On Any Dialogue Started` (Custom Event: `On Any Dialogue Started`, Input: `Instance` UMayDialogueInstance) UND `Bind Event to On Any Dialogue Ended` (Custom Event: `On Any Dialogue Ended`, Input: `Instance`). Beide Custom Events haben einen einfachen `Print String`-Node als Platzhalter.

```text
[Event BeginPlay]
    │
    ▼
[Get MayDialogue Subsystem]
    ├─→ [Bind Event to On Any Dialogue Started] ──► Custom Event: Handle Start
    └─→ [Bind Event to On Any Dialogue Ended]   ──► Custom Event: Handle End
```

#### C++

```cpp
void AQuestDirector::BeginPlay()
{
    Super::BeginPlay();
    if (auto* Sub = GetWorld()->GetSubsystem<UMayDialogueSubsystem>())
    {
        Sub->OnAnyDialogueStarted.AddDynamic(this, &AQuestDirector::HandleDialogueStart);
        Sub->OnAnyDialogueEnded.AddDynamic(this, &AQuestDirector::HandleDialogueEnd);
    }
}

void AQuestDirector::HandleDialogueEnd(UMayDialogueInstance* Instance)
{
    if (!Instance) return;
    UMayDialogueAsset* Asset = Instance->GetDialogueAsset();
    EMayDialogueExitStatus Status = Instance->CurrentExitStatus;

    if (Asset == QuestDialogRef && Status == EMayDialogueExitStatus::Completed)
    {
        AdvanceQuestStep();
    }
}
```

---

## Instance-Delegates (pro Gespräch)

Binde dich an die Instance unmittelbar nach `StartDialogue`. Die Delegates leben so lange wie die Instance.

### OnDialogueStarted

Feuert direkt nachdem die Instance erzeugt wurde, vor dem ersten Node.

**Use-Cases:** Player-Bewegung einfrieren, Quest-Log-Eintrag anlegen, Kamera-Setup.

```cpp
UMayDialogueInstance* Inst = Sub->StartDialogue(Asset, Player, NPC);
if (Inst)
{
    Inst->OnDialogueStarted.AddDynamic(this, &AMyPawn::HandleStart);
}
```

---

### OnDialogueEnded

Feuert am Ende — enthält `ExitStatus` (`Completed`, `Failed`, `Aborted`) und die Laufzeit-Dauer.

**Use-Cases:** Player-Bewegung freigeben, Quest-Abschluss prüfen, Follow-up starten.

> 📸 **Bild-Platzhalter:** `instance-ended-delegate-bp.png` — BP-Graph: Instance von StartDialogue → Bind Event to On Dialogue Ended → Custom Event mit ExitStatus-Branch.
> *Setup:* NPC-Blueprint oder Quest-Director. `Start Dialogue` Return Value → `Bind Event to On Dialogue Ended` (Custom Event: `On Dialogue Ended`, Params: `Asset`, `ExitStatus`, `Duration`, `Instigator`, `Target`). Custom Event hat einen `Switch on EMayDialogueExitStatus`-Node. `Completed`-Zweig triggert `Complete Quest Step`.

```text
[Start Dialogue] → Return Value
    │
    ▼
[Bind Event to On Dialogue Ended]  ──► Custom Event: Handle Dialogue Ended
                                            │ ExitStatus (Enum)
                                            ▼
                                       [Switch on ExitStatus]
                                         ├─ Completed → Quest abschließen
                                         ├─ Failed    → Retry-Logik
                                         └─ Aborted   → Aufräumen
```

---

### OnMessageReceived

Feuert wenn eine SayLine eine Nachricht aufbaut. Das UI hängt sich hier ein.

**Use-Cases:** Eigenes UI-Widget bauen, Logging, Untertitel-System.

```cpp
Inst->OnMessageReceived.AddDynamic(this, &UMyDialogWidget::HandleMessage);

void UMyDialogWidget::HandleMessage(const FMayDialogueMessage& Message)
{
    SpeakerNameText->SetText(Message.DisplayName);
    DialogueText->SetText(Message.Text);
}
```

---

### OnChoicesPresented / OnChoiceMade

`OnChoicesPresented` feuert wenn ein PlayerChoice-Node seine Optionen aufgebaut hat.
`OnChoiceMade` feuert nachdem eine Wahl getroffen wurde (vor der Transition).

**Use-Cases:** Choice-Buttons rendern, Timeout-Bar starten, Analytics.

```cpp
Inst->OnChoicesPresented.AddDynamic(this, &UChoiceWidget::ShowChoices);
Inst->OnChoiceMade.AddDynamic(this, &UAnalytics::LogChoice);

// OnChoiceMade handler must match the 2-param signature:
// void UAnalytics::LogChoice(int32 ChoiceIndex, const FGameplayTagContainer& ChoiceTags);
```

---

### OnVariableChanged

Feuert nach jeder Variablen-Mutation — egal ob durch einen SetVariable-Node oder durch externen Code.

**Use-Cases:** HUD-Update (Dispositions-Anzeige), Live-Quest-Status, Debug-Overlay.

```cpp
Inst->OnVariableChanged.AddDynamic(this, &UDispositionHUD::HandleVarChange);

void UDispositionHUD::HandleVarChange(
    FName VarName,
    EMayDialogueVariableScope Scope,
    EMayDialogueVariableType Type,
    const FString& OldValue,
    const FString& NewValue)
{
    if (VarName == "DispositionLevel")
    {
        int32 Level = FCString::Atoi(*NewValue);
        UpdateDispositionBar(Level);
    }
}
```

{% hint style="info" %}
Both the old and new values are provided as strings. `OldValue` is empty when the variable did not previously exist. Parse values with `Type` (e.g. `FCString::Atoi` for Int).
{% endhint %}

---

### OnDialogueEventFired

Feuert wenn ein FireEvent-Node im Dialog-Graphen einen `FGameplayTag` sendet.

**Use-Cases:** Lose Kopplung zu externen Systemen — Lichteffekte, AI-Zustandsänderungen, Sound-Trigger — ohne direkte Abhängigkeit im Graphen.

```cpp
Inst->OnDialogueEventFired.AddDynamic(this, &ALightController::HandleDialogEvent);

void ALightController::HandleDialogEvent(const FGameplayTag& EventTag)
{
    if (EventTag == TAG_Dialogue_Event_LightsOut)
    {
        TurnOffAllLights();
    }
}
```

---

## Binding-Regeln

- Alle Delegates sind **Multicast** — beliebig viele Listener gleichzeitig.
- Instance-Delegates werden mit der Instance zerstört. Wenn du mehrere Dialoge beobachten willst, nutze die Subsystem-Delegates.
- Subsystem-Delegates leben bis zum Welt-Ende — bei Objekten mit kürzerer Lebenszeit in `EndPlay` wieder `RemoveDynamic` aufrufen.

```cpp
void AMyActor::EndPlay(const EEndPlayReason::Type Reason)
{
    if (auto* Sub = GetWorld()->GetSubsystem<UMayDialogueSubsystem>())
    {
        Sub->OnAnyDialogueStarted.RemoveDynamic(this, &AMyActor::HandleStart);
        Sub->OnAnyDialogueEnded.RemoveDynamic(this, &AMyActor::HandleEnd);
    }
    Super::EndPlay(Reason);
}
```

---

## IMayDialogueBridge (Blueprint oder C++)

`IMayDialogueBridge` ist jetzt vollständig **Blueprintable** — alle 14 Methoden sind `BlueprintNativeEvent` mit C++-Defaults. Du kannst das Interface sowohl in Blueprint als auch in C++ implementieren.

**Blueprint-Weg:** Blueprint-Klasse anlegen → Class Settings → Implemented Interfaces → `MayDialogueBridge` hinzufügen → gewünschte Methoden overriden. Alle Methoden erscheinen im My-Blueprint-Panel unter Interfaces. Details: [Bridge-Implementation](../extension/bridge-implementation.md).

**C++-Weg:** Externe C++-Module (z.B. eigene Plugins) können MayDialogue über das Interface konsumieren, ohne eine harte Abhängigkeit auf das `MayDialogue`-Modul zu brauchen. Das Subsystem implementiert dieses Interface vollständig.

```cpp
// Nur MayDialogueBridge.h importieren, nicht MayDialogueSubsystem.h
IMayDialogueBridge* Bridge = Cast<IMayDialogueBridge>(
    World->GetSubsystem<UMayDialogueSubsystem>());
```

Details zu den Bridge-Read/Write-Methoden: [Read/Write-API](read-write-api.md).
