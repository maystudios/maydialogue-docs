---
description: Dialog-Events in externe Systeme leiten — Quest, Achievement, Analytics per Subsystem-Delegates.
---

# Bridge-Implementation

Wenn ein externes System (Quest-System, Achievement-System, Analytics, eigener Scripting-Layer) auf Dialog-Events reagieren soll, bindet es sich an die Delegates des `UMayDialogueSubsystem`. Dafür ist kein direkter Modul-Link auf Dialog-Internas nötig.

---

## Zwei Integrationswege

| Weg | Wann |
| --- | --- |
| **Subsystem-Delegates binden** | Dein System reagiert auf Dialog-Start, -Ende, Choice, Variablen-Änderung |
| **Bridge-Interface lesen/schreiben** | Dein System liest Dialog-Variablen oder setzt sie (z.B. Hint-Text basierend auf Quest-State) |

---

## Schritt 1 — Subsystem-Delegate binden (Blueprint)

Das einfachste Hookup: im `BeginPlay` deines Systems die Subsystem-Delegates abonnieren.

> 📸 **Bild-Platzhalter:** `bridge-step1-bind-bp.png` — Blueprint-Graph: BeginPlay → Get MayDialogue Subsystem → Bind On Any Dialogue Started.
> *Setup:* BP-Graph im QuestManager-Blueprint. `Event Begin Play` → `Get Game Instance Subsystem` (Class = MayDialogueSubsystem) → `Bind Event to On Any Dialogue Started` (Custom-Event = `OnDialogueStarted`). Zweite Kette: `Bind Event to On Any Dialogue Ended` (Custom-Event = `OnDialogueEnded`). Beide Bindings nebeneinander, alle Pins verbunden.

Verfügbare Subsystem-Delegates:

| Delegate | Wann gefeuert |
| --- | --- |
| `OnAnyDialogueStarted` | Irgendein Dialog startet in der Welt |
| `OnAnyDialogueEnded` | Irgendein Dialog endet |
| `OnChoiceMade` | Spieler wählt eine Choice (per Instance) |
| `OnVariableChanged` | Eine Dialog-Variable oder Participant-Variable ändert sich |

> 📸 **Bild-Platzhalter:** `bridge-step1-events-list.png` — MayDialogueSubsystem im Blueprint-Detail-Panel, Delegates aufgeklappt.
> *Setup:* Im Blueprint-Editor, Subsystem-Variable ausgewählt. Details-Panel rechts zeigt alle `BlueprintAssignable`-Delegates des Subsystems: `OnAnyDialogueStarted`, `OnAnyDialogueEnded`. Beide Einträge sichtbar, Bind-Button für jeden hervorgehoben.

---

## Schritt 2 — Event-Handler implementieren

Für jedes Delegate legst du einen Custom-Event an:

```text
Custom Event: OnDialogueStarted (Asset, Instigator, Target)
  │
  ├─ Is Asset's Tag equal to DA_QuestCritical_Meeting?
  │     └─ true → Quest System: Mark Meeting as Started
  │
  └─ Analytics: Log Event "dialogue_started" (Asset.Name)
```

```text
Custom Event: OnDialogueEnded (Asset, Outcome, Instigator, Target)
  │
  ├─ Is Outcome == Completed?
  │     └─ true → Quest System: Advance Quest
  │
  └─ Analytics: Log Event "dialogue_ended" (Asset.Name, Outcome)
```

> 📸 **Bild-Platzhalter:** `bridge-step2-handler-graph.png` — Custom-Event OnDialogueEnded mit Quest-Advance und Analytics-Log.
> *Setup:* BP-Graph, Custom-Event `OnDialogueEnded (Asset, Outcome, Instigator, Target)`. Von links: Event-Node → `Branch: Outcome == Completed`. True-Pfad: `Get Quest Subsystem → Advance Quest (QuestID="MainStory_MeetNPC")`. False-Pfad: (leer). Nach dem Branch: `Analytics Log (Event="dialogue_ended", Value=Asset.Name)`. Alle Verbindungen sichtbar, klarer Pfad.

---

## Schritt 3 — Per-Instance-Events binden

Wenn du auf eine spezifische Dialog-Instance reagieren willst (nicht global):

```text
Custom Event: OnDialogueStarted (Instance, Asset, Instigator, Target)
  │
  └─ Bind to Instance.OnChoiceMade → Custom-Event OnChoiceMade
```

```text
Custom Event: OnChoiceMade (ChoiceIndex, ChoiceText, Instance)
  │
  └─ Analytics: Log Choice (ChoiceIndex, ChoiceText.ToString())
```

> 📸 **Bild-Platzhalter:** `bridge-step3-instance-bind.png` — Blueprint: OnDialogueStarted → Bind Instance.OnChoiceMade.
> *Setup:* BP-Graph, Custom-Event `OnDialogueStarted`. → `Bind Event to OnChoiceMade` (auf dem Instance-Parameter). Custom-Event `OnChoiceMade` mit Parametern `ChoiceIndex (int), ChoiceText (Text)`. Im Handler: `Analytics Log` mit String-Append "choice:" + ChoiceIndex.

---

## Schritt 4 — Dialog-Variablen lesen und schreiben (Bridge-Interface)

Das `UMayDialogueSubsystem` implementiert `IMayDialogueBridge`. Damit kannst du aus externen Systemen Dialog-Variablen lesen und setzen — ohne direkten Import auf Dialog-Internas.

```text
[Dein Quest-System möchte einen Hint-Text setzen]
  │
  ├─ Get MayDialogue Subsystem
  │
  ├─ Is Dialogue Active?
  │     └─ true →
  │           Set Dialogue Variable
  │             Name:  "QuestHint"
  │             Type:  String
  │             Value: "Suche den Turm im Norden"
  │
  └─ false → (kein aktiver Dialog, nichts tun)
```

```text
[Dein Quest-System möchte prüfen welchen Pfad der Spieler gewählt hat]
  │
  └─ Get Participant Variable
       ParticipantTag: Dialogue.Speaker.Guard
       Name:           "PlayerChoice"
       Type:           String
       → OutValue → auswerten
```

> 📸 **Bild-Platzhalter:** `bridge-step4-read-write-bp.png` — Blueprint-Graph: Is Dialogue Active → Set Dialogue Variable.
> *Setup:* BP-Graph im QuestManager. `Get Game Instance Subsystem (MayDialogueSubsystem)` → `Is Dialogue Active` → Branch. True-Pfad: `Set Dialogue Variable` (Name="QuestHint", Type=String, Value="Suche den Turm"). False-Pfad: `Print String "Kein Dialog aktiv"`. Alle Pins verbunden.

---

## Vollständiges Integrations-Beispiel: Quest + Achievement + Analytics

```text
[BeginPlay]
  │
  ├─ Bind OnAnyDialogueStarted  → HandleStart
  ├─ Bind OnAnyDialogueEnded    → HandleEnd
  └─ (Instance-Events on HandleStart)

[HandleStart (Asset, Instigator, Target)]
  │
  ├─ Log Analytics: dialogue_started (Asset.Name)
  ├─ Bind Instance.OnChoiceMade → HandleChoice
  └─ Quest: Mark this dialogue as "in progress" (Asset.Tag)

[HandleEnd (Asset, Outcome, Instigator, Target)]
  │
  ├─ Log Analytics: dialogue_ended (Asset.Name, Outcome)
  ├─ If Outcome == Completed:
  │     Quest: Advance (QuestID from Asset.Tag)
  │     Achievement: Check if Achievement unlocks
  └─ If Outcome == Failed:
        Quest: Mark dialogue as failed

[HandleChoice (ChoiceIndex, ChoiceText)]
  │
  └─ Log Analytics: dialogue_choice (ChoiceText.ToString())
```

> 📸 **Bild-Platzhalter:** `bridge-full-example-bp.png` — Überblick: drei Custom-Events (HandleStart, HandleEnd, HandleChoice) nebeneinander im BP-Graph.
> *Setup:* BP-Graph-Übersicht mit drei Custom-Event-Blöcken nebeneinander. `HandleStart` links (drei ausgehende Aktionen: Analytics Log, Instance-Bind, Quest-Mark). `HandleEnd` Mitte (Branch auf Outcome, zwei Pfade). `HandleChoice` rechts (ein Analytics-Log). Kommentar-Boxen beschriften jeden Block. Gesamtüberblick des Integration-Patterns sichtbar.

---

## Für C++-Nutzer: In Subsystem-Delegates binden

```cpp
// In deinem Subsystem oder Manager:
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

void UMyQuestManager::HandleDialogueStart(
    UMayDialogueAsset* Asset, AActor* Instigator, AActor* Target)
{
    UAnalyticsLibrary::LogEvent("dialogue_started", Asset->GetName());
}

void UMyQuestManager::HandleDialogueEnd(
    UMayDialogueAsset* Asset, EMayDialogueOutcome Outcome,
    AActor* Instigator, AActor* Target)
{
    if (Outcome == EMayDialogueOutcome::Completed)
    {
        AdvanceQuestForDialogue(Asset);
        CheckAchievements(Asset);
    }
    UAnalyticsLibrary::LogEvent("dialogue_ended", Asset->GetName());
}
```

---

## Bridge-Interface direkt nutzen (C++)

```cpp
// Subsystem holen — implementiert IMayDialogueBridge
UMayDialogueSubsystem* Sub = GetWorld()->GetGameInstance()->GetSubsystem<UMayDialogueSubsystem>();

// Als Bridge casten und Variablen lesen/schreiben
IMayDialogueBridge* Bridge = Sub;  // Subsystem implementiert das Interface

if (Bridge->IsDialogueActive())
{
    FString Hint = FString("Suche den Turm im Norden");
    Bridge->SetDialogueVariable("QuestHint", EMayDialogueVariableType::String, Hint);

    FString PlayerChoice;
    Bridge->GetParticipantVariable(
        FGameplayTag::RequestGameplayTag("Dialogue.Speaker.Guard"),
        "LastTopic",
        EMayDialogueVariableType::String,
        PlayerChoice);
}
```

> 📸 **Bild-Platzhalter:** `bridge-cpp-snippet.png` — Rider-Editor mit C++ Bridge-Integration-Code geöffnet.
> *Setup:* Datei `MyQuestManager.cpp` in Rider. Sichtbar: `HandleDialogueEnd`-Implementierung mit Outcome-Check, Quest-Advance und Analytics-Log. Syntax-Highlighting aktiv, Zeilen 30–55 sichtbar.

---

## Anmerkungen

* **String-basierte Variable-API ist gewollt.** `SetDialogueVariable` und `GetDialogueVariable` nutzen Strings, damit externe Systeme keine compile-time Typen kennen müssen.
* **Bridge-Methoden sind nicht repliziert.** Für Multiplayer kommunizierst du zwischen Systemen über eigene RPCs — die Bridge ist ein lokaler In-Process-Aufruf.
* **Delegates nicht in Destructors vergessen.** Beim Destroy deines Systems die Delegates wieder ausbinden: `DlgSub->OnAnyDialogueStarted.RemoveDynamic(this, &ThisClass::HandleDialogueStart)`.
