---
description: Lifecycle delegates from MayDialogue — when they fire and how to bind.
---

# Bridge & Lifecycle Events

MayDialogue broadcasts on two levels: **instance delegates** for events of a single conversation, and **subsystem delegates** for global "any dialogue is running" events.

## Overview: When Does What Fire?

| Delegate | Level | Fires when... |
|---|---|---|
| `OnDialogueStarted` | Instance | The instance starts, directly before the first Node. |
| `OnDialogueEnded` | Instance | Dialogue ends — regardless of Completed, Failed, or Aborted. |
| `OnNodeReached` | Instance | A Node was successfully executed. |
| `OnMessageReceived` | Instance | A SayLine delivers a message to the UI. |
| `OnChoicesPresented` | Instance | A PlayerChoice Node builds and filters its options. |
| `OnChoiceMade` | Instance | The player (or code) selects a Choice. |
| `OnVariableChanged` | Instance | A dialogue variable or Participant variable changes. |
| `OnDialogueEventFired` | Instance | A FireEvent Node sends a GameplayTag. |
| `OnAnyDialogueStarted` | Subsystem | Every time any dialogue starts. |
| `OnAnyDialogueAborted` | Subsystem | When any dialogue is aborted — fires **before** `OnAnyDialogueEnded`. |
| `OnAnyDialogueEnded` | Subsystem | Every time any dialogue ends (Completed or Aborted). |

---

## Subsystem Delegates (Global)

Use these when you want to react to **all dialogues** — regardless of which asset or NPC is involved.

### OnAnyDialogueAborted

Fires when an instance ends with `ExitStatus == Aborted` — **before** `OnAnyDialogueEnded`.

**Signature:** `Asset, ExitStatus, Duration (float), Instigator (AActor*), Target (AActor*)`

**Typical use cases:**
- Mark interrupted dialogue as "unfinished" in quest log
- Analytics: log an aborted conversation attempt
- Clean up temporary state without triggering a false "Completed"

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
`OnAnyDialogueEnded` is always fired afterward — regardless of Completed or Aborted. If you bind both, you receive `OnAnyDialogueAborted` first, then `OnAnyDialogueEnded`.
{% endhint %}

---

### OnAnyDialogueStarted / OnAnyDialogueEnded

**Typical use cases:**
- Audio ducking: lower music when a dialogue is running
- Switch input mode globally to GameAndUI
- Analytics: log every dialogue start

#### Blueprint

> 📸 **Image placeholder:** `subsystem-delegate-bind-bp.png` — Subsystem delegate binding in BeginPlay.
> *Setup:* GameMode or GameInstance Blueprint. `Event BeginPlay` → `Get MayDialogue Subsystem` → connection to `Bind Event to On Any Dialogue Started` (Custom Event: `On Any Dialogue Started`, Input: `Instance` UMayDialogueInstance) AND `Bind Event to On Any Dialogue Ended` (Custom Event: `On Any Dialogue Ended`, Input: `Instance`). Both Custom Events have a simple `Print String` Node as placeholder.

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

## Instance Delegates (Per Conversation)

Bind to the instance immediately after `StartDialogue`. The delegates live as long as the instance.

### OnDialogueStarted

Fires directly after the instance is created, before the first Node.

**Use cases:** Freeze player movement, create quest log entry, camera setup.

```cpp
UMayDialogueInstance* Inst = Sub->StartDialogue(Asset, Player, NPC);
if (Inst)
{
    Inst->OnDialogueStarted.AddDynamic(this, &AMyPawn::HandleStart);
}
```

---

### OnDialogueEnded

Fires at the end — contains `ExitStatus` (`Completed`, `Failed`, `Aborted`) and the runtime duration.

**Use cases:** Release player movement, check quest completion, start follow-up.

> 📸 **Image placeholder:** `instance-ended-delegate-bp.png` — BP graph: Instance from StartDialogue → Bind Event to On Dialogue Ended → Custom Event with ExitStatus Branch.
> *Setup:* NPC Blueprint or Quest Director. `Start Dialogue` Return Value → `Bind Event to On Dialogue Ended` (Custom Event: `On Dialogue Ended`, Params: `Asset`, `ExitStatus`, `Duration`, `Instigator`, `Target`). Custom Event has a `Switch on EMayDialogueExitStatus` Node. `Completed` branch triggers `Complete Quest Step`.

```text
[Start Dialogue] → Return Value
    │
    ▼
[Bind Event to On Dialogue Ended]  ──► Custom Event: Handle Dialogue Ended
                                            │ ExitStatus (Enum)
                                            ▼
                                       [Switch on ExitStatus]
                                         ├─ Completed → Complete quest
                                         ├─ Failed    → Retry logic
                                         └─ Aborted   → Clean up
```

---

### OnMessageReceived

Fires when a SayLine builds a message. The UI hooks in here.

**Use cases:** Build custom UI widget, logging, subtitle system.

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

`OnChoicesPresented` fires when a PlayerChoice Node has built its options.
`OnChoiceMade` fires after a choice is made (before the transition).

**Use cases:** Render Choice buttons, start timeout bar, analytics.

```cpp
Inst->OnChoicesPresented.AddDynamic(this, &UChoiceWidget::ShowChoices);
Inst->OnChoiceMade.AddDynamic(this, &UAnalytics::LogChoice);

// OnChoiceMade handler must match the 2-param signature:
// void UAnalytics::LogChoice(int32 ChoiceIndex, const FGameplayTagContainer& ChoiceTags);
```

---

### OnVariableChanged

Fires after every variable mutation — whether by a SetVariable Node or external code.

**Use cases:** HUD update (disposition display), live quest status, debug overlay.

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

Fires when a FireEvent Node in the dialogue graph sends an `FGameplayTag`.

**Use cases:** Loose coupling to external systems — light effects, AI state changes, sound triggers — without direct dependency in the graph.

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

## Binding Rules

- All delegates are **multicast** — any number of listeners simultaneously.
- Instance delegates are destroyed with the instance. If you want to observe multiple dialogues, use the subsystem delegates.
- Subsystem delegates live until the world ends — for objects with a shorter lifetime, call `RemoveDynamic` again in `EndPlay`.

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

## IMayDialogueBridge (Blueprint or C++)

### What is the Bridge Interface for?

`IMayDialogueBridge` is the generic interface for external systems — Quest Manager, Analytics, Cheat Menus, MayFlowGraph — that want to control or query MayDialogue without a direct dependency on the subsystem. The subsystem implements the interface fully. The most useful methods are `StartDialogueFromBridge`, `IsDialogueActive`, `AbortDialogue`, `GetDialogueVariable` / `SetDialogueVariable`, `GetPendingChoices`, and `SelectChoice`. The full method overview can be found at [Read/Write API](read-write-api.md).

The interface is fully **Blueprintable** — all methods are `BlueprintNativeEvent` with C++ defaults. You can implement it in both Blueprint and C++.

**Blueprint approach:** Create a Blueprint class → Class Settings → Implemented Interfaces → add `MayDialogueBridge` → override desired methods. All methods appear in the My Blueprint panel under Interfaces. Details: [Bridge Implementation](../extension/bridge-implementation.md).

**C++ approach:** External C++ modules (e.g. your own plugins) can consume MayDialogue via the interface without a hard dependency on the `MayDialogue` module. The subsystem implements this interface fully.

```cpp
// Only import MayDialogueBridge.h, not MayDialogueSubsystem.h
IMayDialogueBridge* Bridge = Cast<IMayDialogueBridge>(
    World->GetSubsystem<UMayDialogueSubsystem>());
```

Details on Bridge read/write methods: [Read/Write API](read-write-api.md).
