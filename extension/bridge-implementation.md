---
description: Dialog-Events in externe Systeme leiten — Quest, Achievement, Analytics per Subsystem-Delegates oder per Bridge-Interface in Blueprint oder C++.
---

# Bridge-Implementation

Wenn ein externes System (Quest-System, Achievement-System, Analytics, eigener Scripting-Layer) auf Dialog-Events reagieren soll, gibt es drei Integrationswege — alle drei funktionieren jetzt vollständig in Blueprint.

---

## Drei Integrationswege

| Weg | Wann |
| --- | --- |
| **Subsystem-Delegates binden** | Dein System reagiert auf Dialog-Start, -Ende, Choice, Variablen-Änderung |
| **Bridge-Interface in Blueprint implementieren** | Dein System liest Dialog-Variablen oder setzt sie — als Blueprint-Klasse, keine C++-Kenntnisse nötig |
| **Bridge-Interface in C++ implementieren** | Wie oben, aber mit voller IDE-Unterstützung und komplexer Subsystem-Logik |

---

## Schritt 1 — Subsystem-Delegate binden (Blueprint)

Das einfachste Hookup: im `BeginPlay` deines Systems die Subsystem-Delegates abonnieren.

> 📸 **Bild-Platzhalter:** `bridge-step1-bind-bp.png` — Blueprint-Graph: BeginPlay → Get MayDialogue Subsystem → Bind On Any Dialogue Started.
> *Setup:* BP-Graph im QuestManager-Blueprint. `Event Begin Play` → `Get Game Instance Subsystem` (Class = MayDialogueSubsystem) → `Bind Event to On Any Dialogue Started` (Custom-Event = `OnDialogueStarted`). Zweite Kette: `Bind Event to On Any Dialogue Ended` (Custom-Event = `OnDialogueEnded`). Beide Bindings nebeneinander, alle Pins verbunden.

Verfügbare Subsystem-Delegates:

| Delegate | Wann gefeuert |
| --- | --- |
| `OnAnyDialogueStarted` | Irgendein Dialog startet in der Welt |
| `OnAnyDialogueEnded` | Irgendein Dialog endet (Completed oder Aborted) |
| `OnAnyDialogueAborted` | Irgendein Dialog wird abgebrochen — feuert **vor** `OnAnyDialogueEnded` |
| `OnChoiceMade` | Spieler wählt eine Choice (per Instance) |
| `OnVariableChanged` | Eine Dialog-Variable oder Participant-Variable ändert sich |

> 📸 **Bild-Platzhalter:** `bridge-step1-events-list.png` — MayDialogueSubsystem im Blueprint-Detail-Panel, Delegates aufgeklappt.
> *Setup:* Im Blueprint-Editor, Subsystem-Variable ausgewählt. Details-Panel rechts zeigt alle `BlueprintAssignable`-Delegates des Subsystems: `OnAnyDialogueStarted`, `OnAnyDialogueEnded`, `OnAnyDialogueAborted`. Alle Einträge sichtbar, Bind-Button für jeden hervorgehoben.

---

## Schritt 2 — Event-Handler implementieren

Für jedes Delegate legst du einen Custom-Event an. Die Subsystem-Delegates `OnAnyDialogueStarted` und `OnAnyDialogueEnded` übergeben nur die **Instance** — Asset, Instigator und Target liest du bei Bedarf von der Instance ab:

```text
Custom Event: OnDialogueStarted (Instance)
  │
  ├─ Get Dialogue Asset von Instance
  ├─ Is Asset's Tag equal to DA_QuestCritical_Meeting?
  │     └─ true → Quest System: Mark Meeting as Started
  │
  └─ Analytics: Log Event "dialogue_started" (Asset.Name)
```

```text
Custom Event: OnDialogueEnded (Instance)
  │
  ├─ Get Exit Status von Instance
  ├─ Is Exit Status == Completed?
  │     └─ true → Quest System: Advance Quest
  │
  └─ Analytics: Log Event "dialogue_ended" (Instance.GetDialogueAsset.Name)
```

> 📸 **Bild-Platzhalter:** `bridge-step2-handler-graph.png` — Custom-Event OnDialogueEnded mit Quest-Advance und Analytics-Log.
> *Setup:* BP-Graph, Custom-Event `OnDialogueEnded (Instance)`. Von links: Event-Node → `Get Dialogue Asset` → `Branch: ExitStatus == Completed`. True-Pfad: `Get Quest Subsystem → Advance Quest`. False-Pfad: (leer). Nach dem Branch: `Analytics Log`. Alle Verbindungen sichtbar.

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

## Bridge-Interface in Blueprint implementieren (BP-First-Weg)

`IMayDialogueBridge` ist vollständig `Blueprintable`. Du kannst eine eigene Blueprint-Klasse anlegen, die das Interface implementiert, und gezielt nur die Methoden überschreiben, die dein System braucht. Alle Methoden sind `BlueprintNativeEvent` mit C++-Defaults — du musst nur das überschreiben, was du anpassen willst.

### Anwendungsfall: Quest-Bridge

Ziel: Eine Blueprint-Klasse `BP_QuestBridge`, die reagiert wenn ein Dialog startet, und den aktuellen Quest-Hint in die Dialog-Variable `QuestHint` schreibt.

**Schritt A — Blueprint-Klasse anlegen:**

1. Content Browser → Rechtsklick → **Blueprint Class**
2. Im Klassen-Picker **All Classes** öffnen
3. `MayDialogueSubsystem` suchen (die Subsystem-Klasse implementiert `IMayDialogueBridge` als Default) — **oder** du leitest von `UObject` ab und implementierst das Interface manuell über **Class Settings → Implemented Interfaces → Add → MayDialogueBridge**
4. Benennen: `BP_QuestBridge`

> 📸 **Bild-Platzhalter:** `bridge-bp-interface-add.png` — Class Settings im BP-Editor, "Implemented Interfaces"-Panel mit MayDialogueBridge in der Liste.
> *Setup:* BP-Editor offen (`BP_QuestBridge`). Oben: `Class Settings`. Rechts: `Interfaces`-Panel. `Implemented Interfaces`-Liste zeigt `MayDialogueBridge` als Eintrag. `Add`-Button darunter sichtbar.

**Schritt B — `On Dialogue Started` überschreiben:**

Im **My Blueprint**-Panel unter Interfaces → `On Dialogue Started` → Doppelklick um den Override-Graphen zu öffnen.

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

> 📸 **Bild-Platzhalter:** `bridge-bp-ondialoguestarted.png` — Override-Graph von On Dialogue Started mit Quest-Update und Set Dialogue Variable.
> *Setup:* BP-Graph, `Event On Dialogue Started (Asset, Instigator, Target)`. Zwei parallele Pfade: 1. `Get Quest Subsystem → Mark Dialogue In Progress (Asset.Tag)`. 2. `Get Quest Hint From Quest Subsystem` → `Set Dialogue Variable` (Name="QuestHint", Type=String, Value=Quest-Hint-String). Alle Pins verbunden.

**Schritt C — Weitere Methoden überschreiben (optional):**

Alle Methoden des Interface sind im My Blueprint-Panel sichtbar. Override nur was du brauchst:

| Methode (Blueprint-Name) | Typischer Einsatz |
| --- | --- |
| `On Dialogue Started` | Quest-Tracking, Analytics-Start |
| `On Dialogue Ended` | Quest-Advance, Achievement-Unlock |
| `Is Dialogue Active` | Status aus eigenem System lesen |
| `Abort Dialogue` | Dialog von außen abbrechen |
| `Can Start Dialogue` | Eigene Start-Precondition einbauen |
| `Get Active Dialogue Asset` | Asset von außen lesen |
| `Get Current Node GUID` | Node-Tracking für Analytics |
| `Get Active Participants` | Participant-Array lesen |
| `Get Dialogue Variable` | Variable lesen |
| `Get Participant Variable` | Participant-Variable lesen |
| `Get Pending Choices` | Choices von außen lesen |
| `Set Dialogue Variable` | Variable setzen (z.B. Quest-Hint) |
| `Set Participant Variable` | Participant-Variable setzen |
| `Select Choice` | Choice programmatisch auswählen |
| `Force Advance` | Advance überspringen |

{% hint style="info" %}
**Alle Methoden haben C++-Defaults.** Du überschreibst nur, was du brauchst. Methoden die du nicht overridest, delegieren automatisch an die Basisimplementierung des Subsystems.
{% endhint %}

---

## Schritt 4 — Dialog-Variablen lesen und schreiben

Das `UMayDialogueSubsystem` implementiert `IMayDialogueBridge`. Damit kannst du aus externen Systemen Dialog-Variablen lesen und setzen — direkt aus Blueprint ohne C++-Code.

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
  ├─ Bind OnAnyDialogueAborted  → HandleAbort
  └─ (Instance-Events on HandleStart)

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
  ├─ Get ExitStatus von Instance
  ├─ If ExitStatus == Completed:
  │     Quest: Advance (QuestID from Instance.Asset.Tag)
  │     Achievement: Check if Achievement unlocks
  └─ If ExitStatus == Failed:
        Quest: Mark dialogue as failed

[HandleChoice (ChoiceIndex, ChoiceText)]
  │
  └─ Log Analytics: dialogue_choice (ChoiceText.ToString())
```

> 📸 **Bild-Platzhalter:** `bridge-full-example-bp.png` — Überblick: vier Custom-Events (HandleStart, HandleAbort, HandleEnd, HandleChoice) nebeneinander im BP-Graph.
> *Setup:* BP-Graph-Übersicht mit vier Custom-Event-Blöcken nebeneinander. `HandleStart` links (drei ausgehende Aktionen: Analytics Log, Instance-Bind, Quest-Mark). `HandleAbort` (Analytics + Quest-Mark). `HandleEnd` Mitte (Branch auf Outcome, zwei Pfade). `HandleChoice` rechts (ein Analytics-Log). Kommentar-Boxen beschriften jeden Block. Gesamtüberblick des Integration-Patterns sichtbar.

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

// OnAnyDialogueStarted / OnAnyDialogueEnded: Signatur ist (UMayDialogueInstance* Instance)
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

* **Blueprint und C++ gleichwertig.** `IMayDialogueBridge` ist `Blueprintable`, alle Methoden sind `BlueprintNativeEvent`. Wähle den Weg, der zu deinem Projekt passt.
* **`OnAnyDialogueAborted` feuert vor `OnAnyDialogueEnded`.** Wenn du beide bindest, erhältst du zuerst `Aborted`, dann immer `Ended`.
* **String-basierte Variable-API ist gewollt.** `SetDialogueVariable` und `GetDialogueVariable` nutzen Strings, damit externe Systeme keine compile-time Typen kennen müssen.
* **Bridge-Methoden sind nicht repliziert.** Für Multiplayer kommunizierst du zwischen Systemen über eigene RPCs — die Bridge ist ein lokaler In-Process-Aufruf.
* **Delegates nicht in Destructors vergessen.** Beim Destroy deines Systems die Delegates wieder ausbinden: `DlgSub->OnAnyDialogueStarted.RemoveDynamic(this, &ThisClass::HandleDialogueStart)`.
