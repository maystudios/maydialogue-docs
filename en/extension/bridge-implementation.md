---
description: Route dialogue events to external systems — Quest, Achievement, Analytics via Subsystem delegates or the Bridge interface in Blueprint or C++.
---

# Bridge Implementation

When an external system (quest system, achievement system, analytics, custom scripting layer) needs to react to dialogue events, there are three integration paths — all three work fully in Blueprint.

---

## Three Integration Paths

| Path | When |
| --- | --- |
| **Bind Subsystem Delegates** | Your system reacts to dialogue start, end, choice, variable change |
| **Implement Bridge Interface in Blueprint** | Your system reads or writes dialogue variables — as a Blueprint class, no C++ knowledge required |
| **Implement Bridge Interface in C++** | Same as above, but with full IDE support and complex subsystem logic |

---

## Step 1 — Bind Subsystem Delegate (Blueprint)

The simplest hookup: subscribe to the Subsystem delegates in your system's `BeginPlay`.

> 📸 **Image placeholder:** `bridge-step1-bind-bp.png` — Blueprint graph: BeginPlay → Get MayDialogue Subsystem → Bind On Any Dialogue Started.
> *Setup:* Blueprint graph in the QuestManager Blueprint. `Event Begin Play` → `Get Game Instance Subsystem` (Class = MayDialogueSubsystem) → `Bind Event to On Any Dialogue Started` (Custom Event = `OnDialogueStarted`). Second chain: `Bind Event to On Any Dialogue Ended` (Custom Event = `OnDialogueEnded`). Both bindings side by side, all pins connected.

Available Subsystem delegates:

| Delegate | When Fired |
| --- | --- |
| `OnAnyDialogueStarted` | Any dialogue starts in the world |
| `OnAnyDialogueEnded` | Any dialogue ends (Completed or Aborted) |
| `OnAnyDialogueAborted` | Any dialogue is aborted — fires **before** `OnAnyDialogueEnded` |
| `OnChoiceMade` | Player selects a choice (per instance) |
| `OnVariableChanged` | A dialogue variable or participant variable changes |

> 📸 **Image placeholder:** `bridge-step1-events-list.png` — MayDialogueSubsystem in the Blueprint Details panel, delegates expanded.
> *Setup:* In the Blueprint editor, Subsystem variable selected. Details panel on the right shows all `BlueprintAssignable` delegates of the Subsystem: `OnAnyDialogueStarted`, `OnAnyDialogueEnded`, `OnAnyDialogueAborted`. All entries visible, Bind button highlighted for each.

---

## Step 2 — Implement Event Handlers

For each delegate, create a Custom Event. The Subsystem delegates `OnAnyDialogueStarted` and `OnAnyDialogueEnded` only pass the **Instance** — you read the Asset, Instigator, and Target from the Instance as needed:

```text
Custom Event: OnDialogueStarted (Instance)
  │
  ├─ Get Dialogue Asset from Instance
  ├─ Is Asset's Tag equal to DA_QuestCritical_Meeting?
  │     └─ true → Quest System: Mark Meeting as Started
  │
  └─ Analytics: Log Event "dialogue_started" (Asset.Name)
```

```text
Custom Event: OnDialogueEnded (Instance)
  │
  ├─ Get Exit Status from Instance
  ├─ Is Exit Status == Completed?
  │     └─ true → Quest System: Advance Quest
  │
  └─ Analytics: Log Event "dialogue_ended" (Instance.GetDialogueAsset.Name)
```

> 📸 **Image placeholder:** `bridge-step2-handler-graph.png` — Custom Event OnDialogueEnded with Quest Advance and Analytics Log.
> *Setup:* Blueprint graph, Custom Event `OnDialogueEnded (Instance)`. From left: Event node → `Get Dialogue Asset` → `Branch: ExitStatus == Completed`. True path: `Get Quest Subsystem → Advance Quest`. False path: (empty). After the branch: `Analytics Log`. All connections visible.

---

## Step 3 — Bind Per-Instance Events

When you want to react to a specific dialogue instance (not globally):

```text
Custom Event: OnDialogueStarted (Instance, Asset, Instigator, Target)
  │
  └─ Bind to Instance.OnChoiceMade → Custom Event OnChoiceMade
```

```text
Custom Event: OnChoiceMade (ChoiceIndex, ChoiceText, Instance)
  │
  └─ Analytics: Log Choice (ChoiceIndex, ChoiceText.ToString())
```

> 📸 **Image placeholder:** `bridge-step3-instance-bind.png` — Blueprint: OnDialogueStarted → Bind Instance.OnChoiceMade.
> *Setup:* Blueprint graph, Custom Event `OnDialogueStarted`. → `Bind Event to OnChoiceMade` (on the Instance parameter). Custom Event `OnChoiceMade` with parameters `ChoiceIndex (int), ChoiceText (Text)`. In the handler: `Analytics Log` with string append "choice:" + ChoiceIndex.

---

## Implement Bridge Interface in Blueprint (Blueprint-First Path)

`IMayDialogueBridge` is fully `Blueprintable`. You can create your own Blueprint class that implements the interface and selectively override only the methods your system needs. All methods are `BlueprintNativeEvent` with C++ defaults — you only override what you want to customize.

### Use Case: Quest Bridge

Goal: A Blueprint class `BP_QuestBridge` that reacts when a dialogue starts and writes the current quest hint into the dialogue variable `QuestHint`.

**Step A — Create Blueprint Class:**

1. Content Browser → right-click → **Blueprint Class**
2. Open **All Classes** in the class picker
3. Search for `MayDialogueSubsystem` (the Subsystem class implements `IMayDialogueBridge` by default) — **or** derive from `UObject` and implement the interface manually via **Class Settings → Implemented Interfaces → Add → MayDialogueBridge**
4. Name it: `BP_QuestBridge`

> 📸 **Image placeholder:** `bridge-bp-interface-add.png` — Class Settings in the Blueprint editor, "Implemented Interfaces" panel with MayDialogueBridge in the list.
> *Setup:* Blueprint editor open (`BP_QuestBridge`). At the top: `Class Settings`. On the right: `Interfaces` panel. `Implemented Interfaces` list shows `MayDialogueBridge` as an entry. `Add` button visible below.

**Step B — Override `On Dialogue Started`:**

In the **My Blueprint** panel under Interfaces → `On Dialogue Started` → double-click to open the override graph.

```text
Event On Dialogue Started (Asset, Instigator, Target)
  │
  ├─ Quest System: Mark dialogue "in progress" (Asset.Tag)
  │
  └─ Set Dialogue Variable
       Name: "QuestHint"
       Type: String
       Value: Get Current Quest Hint (from Quest Subsystem)
```

> 📸 **Image placeholder:** `bridge-bp-ondialoguestarted.png` — Override graph for On Dialogue Started with Quest update and Set Dialogue Variable.
> *Setup:* Blueprint graph, `Event On Dialogue Started (Asset, Instigator, Target)`. Two parallel paths: 1. `Get Quest Subsystem → Mark Dialogue In Progress (Asset.Tag)`. 2. `Get Quest Hint From Quest Subsystem` → `Set Dialogue Variable` (Name="QuestHint", Type=String, Value=Quest hint string). All pins connected.

**Step C — Override additional methods (optional):**

All interface methods are visible in the My Blueprint panel. Override only what you need:

| Method (Blueprint Name) | Typical Use |
| --- | --- |
| `On Dialogue Started` | Quest tracking, analytics start |
| `On Dialogue Ended` | Quest advance, achievement unlock |
| `Is Dialogue Active` | Read status from your own system |
| `Abort Dialogue` | Abort dialogue from outside |
| `Can Start Dialogue` | Add a custom start precondition |
| `Get Active Dialogue Asset` | Read asset from outside |
| `Get Current Node GUID` | Node tracking for analytics |
| `Get Active Participants` | Read participant array |
| `Get Dialogue Variable` | Read variable |
| `Get Participant Variable` | Read participant variable |
| `Get Pending Choices` | Read choices from outside |
| `Set Dialogue Variable` | Set variable (e.g. quest hint) |
| `Set Participant Variable` | Set participant variable |
| `Select Choice` | Select choice programmatically |
| `Force Advance` | Skip advance |

{% hint style="info" %}
**All methods have C++ defaults.** You only override what you need. Methods you don't override automatically delegate to the Subsystem's base implementation.
{% endhint %}

---

## Step 4 — Read and Write Dialogue Variables

`UMayDialogueSubsystem` implements `IMayDialogueBridge`. This allows external systems to read and set dialogue variables — directly from Blueprint without C++ code.

```text
[Your Quest System wants to set a hint text]
  │
  ├─ Get MayDialogue Subsystem
  │
  ├─ Is Dialogue Active?
  │     └─ true →
  │           Set Dialogue Variable
  │             Name:  "QuestHint"
  │             Type:  String
  │             Value: "Find the tower in the north"
  │
  └─ false → (no active dialogue, do nothing)
```

```text
[Your Quest System wants to check which path the player chose]
  │
  └─ Get Participant Variable
       ParticipantTag: Dialogue.Speaker.Guard
       Name:           "PlayerChoice"
       Type:           String
       → OutValue → evaluate
```

> 📸 **Image placeholder:** `bridge-step4-read-write-bp.png` — Blueprint graph: Is Dialogue Active → Set Dialogue Variable.
> *Setup:* Blueprint graph in QuestManager. `Get Game Instance Subsystem (MayDialogueSubsystem)` → `Is Dialogue Active` → Branch. True path: `Set Dialogue Variable` (Name="QuestHint", Type=String, Value="Find the tower"). False path: `Print String "No dialogue active"`. All pins connected.

---

## Complete Integration Example: Quest + Achievement + Analytics

```text
[BeginPlay]
  │
  ├─ Bind OnAnyDialogueStarted  → HandleStart
  ├─ Bind OnAnyDialogueEnded    → HandleEnd
  ├─ Bind OnAnyDialogueAborted  → HandleAbort
  └─ (Instance events in HandleStart)

[HandleStart (Instance)]
  │
  ├─ Log Analytics: dialogue_started (Instance.Asset.Name)
  ├─ Bind Instance.OnChoiceMade → HandleChoice
  └─ Quest: Mark this dialogue as "in progress" (Instance.Asset.Tag)

[HandleAbort (Instance)]
  │
  ├─ Log Analytics: dialogue_aborted (Instance.Asset.Name)
  └─ Quest: Mark dialogue as interrupted

[HandleEnd (Instance)]
  │
  ├─ Get ExitStatus from Instance
  ├─ If ExitStatus == Completed:
  │     Quest: Advance (QuestID from Instance.Asset.Tag)
  │     Achievement: Check if Achievement unlocks
  └─ If ExitStatus == Failed:
        Quest: Mark dialogue as failed

[HandleChoice (ChoiceIndex, ChoiceText)]
  │
  └─ Log Analytics: dialogue_choice (ChoiceText.ToString())
```

> 📸 **Image placeholder:** `bridge-full-example-bp.png` — Overview: four Custom Events (HandleStart, HandleAbort, HandleEnd, HandleChoice) side by side in the Blueprint graph.
> *Setup:* Blueprint graph overview with four Custom Event blocks side by side. `HandleStart` on the left (three outgoing actions: Analytics Log, Instance Bind, Quest Mark). `HandleAbort` (Analytics + Quest Mark). `HandleEnd` center (Branch on Outcome, two paths). `HandleChoice` on the right (one Analytics Log). Comment boxes label each block. Full overview of the integration pattern visible.

---

## For C++ Users: Binding to Subsystem Delegates

```cpp
// In your Subsystem or Manager:
void UMyQuestManager::Initialize(FSubsystemCollectionBase& Collection)
{
    Super::Initialize(Collection);

    UMayDialogueSubsystem* DlgSub = GetGameInstance()->GetSubsystem<UMayDialogueSubsystem>();
    if (DlgSub)
    {
        DlgSub->OnAnyDialogueStarted.AddDynamic(this, &UMyQuestManager::HandleDialogueStart);
        DlgSub->OnAnyDialogueEnded.AddDynamic(this, &UMyQuestManager::HandleDialogueEnd);
    }
}

// OnAnyDialogueStarted / OnAnyDialogueEnded: signature is (UMayDialogueInstance* Instance)
void UMyQuestManager::HandleDialogueStart(UMayDialogueInstance* Instance)
{
    if (Instance && Instance->GetDialogueAsset())
    {
        UAnalyticsLibrary::LogEvent("dialogue_started", Instance->GetDialogueAsset()->GetName());
    }
}

void UMyQuestManager::HandleDialogueEnd(UMayDialogueInstance* Instance)
{
    if (Instance && Instance->GetDialogueAsset())
    {
        AdvanceQuestForDialogue(Instance->GetDialogueAsset());
        CheckAchievements(Instance->GetDialogueAsset());
        UAnalyticsLibrary::LogEvent("dialogue_ended", Instance->GetDialogueAsset()->GetName());
    }
}
```

---

## Using the Bridge Interface Directly (C++)

```cpp
// Get the Subsystem — it implements IMayDialogueBridge
UMayDialogueSubsystem* Sub = GetWorld()->GetGameInstance()->GetSubsystem<UMayDialogueSubsystem>();

// Cast as Bridge and read/write variables
IMayDialogueBridge* Bridge = Sub;  // Subsystem implements the interface

if (Bridge->IsDialogueActive())
{
    FString Hint = FString("Find the tower in the north");
    Bridge->SetDialogueVariable("QuestHint", EMayDialogueVariableType::String, Hint);

    FString PlayerChoice;
    Bridge->GetParticipantVariable(
        FGameplayTag::RequestGameplayTag("Dialogue.Speaker.Guard"),
        "LastTopic",
        EMayDialogueVariableType::String,
        PlayerChoice);
}
```

> 📸 **Image placeholder:** `bridge-cpp-snippet.png` — Rider editor with C++ Bridge integration code open.
> *Setup:* File `MyQuestManager.cpp` in Rider. Visible: `HandleDialogueEnd` implementation with outcome check, quest advance, and analytics log. Syntax highlighting active, lines 30–55 visible.

---

## Notes

* **Blueprint and C++ are equally valid.** `IMayDialogueBridge` is `Blueprintable`, all methods are `BlueprintNativeEvent`. Choose the path that fits your project.
* **`OnAnyDialogueAborted` fires before `OnAnyDialogueEnded`.** If you bind both, you receive `Aborted` first, then always `Ended`.
* **The string-based variable API is intentional.** `SetDialogueVariable` and `GetDialogueVariable` use strings so external systems don't need to know compile-time types.
* **Bridge methods are not replicated.** For multiplayer, communicate between systems using your own RPCs — the Bridge is a local in-process call.
* **Don't forget to unbind delegates in destructors.** When your system is destroyed, remove the delegates: `DlgSub->OnAnyDialogueStarted.RemoveDynamic(this, &ThisClass::HandleDialogueStart)`.
