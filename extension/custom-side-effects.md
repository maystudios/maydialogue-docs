---
description: Eigene SideEffects per Blueprint — am Beispiel "Quest-Progress-Update".
---

# Eigene SideEffects

SideEffects sind Inline-Aktionen, die beim Betreten eines Nodes ausgeführt werden. Für projekt-spezifische Aktionen — Quest-Update, Inventar-Änderung, Achievement, Analytics-Event — erstellst du eigene Subklassen.

Beispiel-Ziel: ein SideEffect `BP_SE_QuestProgressUpdate`, der einen Quest-Fortschritt um einen konfigurierbaren Wert erhöht.

---

## Schritt 1 — Blueprint-Klasse anlegen

1. Content Browser → Rechtsklick → **Blueprint Class**.
2. **All Classes** öffnen.
3. `MayDialogueSideEffect` suchen und auswählen.
4. Benennen: `BP_SE_QuestProgressUpdate`.

> 📸 **Bild-Platzhalter:** `cse-step1-picker.png` — "Pick Parent Class"-Dialog mit `MayDialogueSideEffect`.
> *Setup:* Blueprint-Klassen-Dialog, "All Classes"-Tab. Suchfeld: "MayDialogueSideEffect". Ergebnis: `UMayDialogueSideEffect` mit vollständigem Pfad. Select-Button aktiv, Mauszeiger auf dem Eintrag.

---

## Schritt 2 — Properties anlegen

Variables-Panel:

* `QuestID` — Typ `FName`, **Instance Editable = true**, Category = "Quest"
* `ProgressDelta` — Typ `int32`, **Instance Editable = true**, Standardwert = 1, Category = "Quest"

> 📸 **Bild-Platzhalter:** `cse-step2-variables.png` — Variables-Panel mit `QuestID` und `ProgressDelta`.
> *Setup:* Blueprint-Editor. Variables-Panel zeigt `QuestID` (Name) und `ProgressDelta` (Integer, Default=1). Details-Panel rechts: `Instance Editable = true`, `Category = Quest`. Beide Variablen sichtbar.

---

## Schritt 3 — ExecuteSideEffect überschreiben

Functions-Panel → **Override** → `Execute Side Effect`.

Pseudo-Graph:

```text
Event Execute Side Effect (Context)
  │
  ├─ Is QuestID valid? (None-Check)
  │     └─ false → Return (früh abbrechen)
  │
  ├─ Get World (Context → DialogueInstance → Get World)
  │
  ├─ Get Quest Subsystem
  │
  └─ Add Quest Progress (QuestID, ProgressDelta)
```

> 📸 **Bild-Platzhalter:** `cse-step3-graph.png` — Vollständiger Execute-Side-Effect-Graph.
> *Setup:* `Event Execute Side Effect (Context)` → `Is Valid Check (QuestID != None)` → Branch. False-Pfad: `Return`. True-Pfad: `Get World` → `Get Quest Subsystem` → `Add Quest Progress` mit gepinnten `QuestID` und `ProgressDelta` Variablen. Alle Exec-Pins verbunden, kein offener Pfad.

{% hint style="info" %}
**Früh abbrechen bei ungültigen Werten.** Prüfe am Anfang, ob alle benötigten Werte gesetzt sind (QuestID != None, ProgressDelta != 0). Ein No-Op ist besser als ein Crash oder unerwartet ausgeführte Logik.
{% endhint %}

---

## Schritt 4 — Im Dialog verwenden

1. Dialog-Asset öffnen.
2. Einen Node (z.B. SayLine) auswählen → Details-Panel → SideEffects-Array → **+** → `BP_SE_QuestProgressUpdate`.
3. `QuestID` eintragen, `ProgressDelta` setzen.
4. Kompilieren und testen.

> 📸 **Bild-Platzhalter:** `cse-step4-in-graph.png` — SayLine mit QuestProgressUpdate-Pill im SideEffects-Array.
> *Setup:* MayDialogue-Editor, SayLine "Der Drache ist gefallen." ausgewählt. SideEffects-Array aufgeklappt, ein Eintrag: `BP_SE_QuestProgressUpdate`. Darunter: `QuestID = KillDragon`, `ProgressDelta = 1`. Pill im Graphen des Nodes zeigt "QuestProgress: KillDragon +1".

---

## Mehrere SideEffects kombinieren

Auf einem Node kannst du beliebig viele SideEffects hintereinander legen — sie laufen in Array-Reihenfolge:

```text
SayLine "Du hast dich bewiesen."
  SideEffects:
    1. Quest Progress Update   QuestID=ProveYourself   ProgressDelta=1
    2. Grant Achievement       AchievementID=Proven
    3. Add Gameplay Tag        TagToAdd=Story.Proven    bAddToInstigator=true
    4. Trigger Gameplay Cue   CueTag=GameplayCue.UI.Achievement
```

> 📸 **Bild-Platzhalter:** `cse-stacked-sideeffects.png` — SayLine mit vier gestapelten SideEffects im Details-Panel.
> *Setup:* MayDialogue-Editor, SayLine "Du hast dich bewiesen." ausgewählt. SideEffects-Array zeigt vier Einträge nummeriert: 1. `QuestProgressUpdate (ProveYourself, +1)`, 2. `GrantAchievement (Proven)`, 3. `AddTag (Story.Proven)`, 4. `TriggerCue (GameplayCue.UI.Achievement)`. Alle Felder ausgefüllt, Array-Index sichtbar.

---

## Server/Client-Split für Multiplayer

`UMayDialogueSideEffect` hat zwei überschreibbare Methoden:

| Methode | Wann aufgerufen |
| --- | --- |
| `Execute Side Effect` | Auf dem Server (Gameplay-Logik) |
| `Execute Client Side Effect` | Auf jedem Client (visuelle Effekte, Sounds) |

Für Singleplayer-Projekte laufen beide immer. Für Multiplayer: Quest-Progress, Inventar-Änderungen → `Execute Side Effect`. Sounds, Particles → `Execute Client Side Effect`.

---

## Typische projekt-spezifische SideEffects

| SideEffect | Aktion |
| --- | --- |
| `SE_QuestProgressUpdate(QuestID, Delta)` | Quest-Fortschritt erhöhen |
| `SE_GrantAchievement(AchievementID)` | Achievement freischalten |
| `SE_Inventory_Add(ItemID, Count)` | Item hinzufügen |
| `SE_Inventory_Remove(ItemID, Count)` | Item entfernen |
| `SE_NPCReputation(FactionTag, Delta)` | Reputationswert verändern |
| `SE_UnlockDoor(DoorTag)` | Tür öffnen |
| `SE_NotifyAnalytics(EventName)` | Analytics-Event senden |

Jedes davon ist eine kleine, fokussierte Blueprint-Klasse.

---

{% hint style="success" %}
**Variante in C++**

```cpp
UCLASS(BlueprintType, EditInlineNew, meta=(DisplayName="Quest Progress Update"))
class MYGAME_API UMySE_QuestProgressUpdate : public UMayDialogueSideEffect
{
    GENERATED_BODY()
public:
    UPROPERTY(EditAnywhere, Category="Quest")
    FName QuestID;

    UPROPERTY(EditAnywhere, Category="Quest")
    int32 ProgressDelta = 1;

    virtual void ExecuteSideEffect_Implementation(const FMayDialogueContext& Context) override;
    virtual FText GetDisplayDescription_Implementation() const override;
};
```

```cpp
void UMySE_QuestProgressUpdate::ExecuteSideEffect_Implementation(const FMayDialogueContext& Context)
{
    if (QuestID == NAME_None || ProgressDelta == 0) return;

    auto* Quest = UQuestSubsystem::Get(Context.DialogueInstance->GetWorld());
    if (Quest)
    {
        Quest->AddProgress(QuestID, ProgressDelta);
    }
}

FText UMySE_QuestProgressUpdate::GetDisplayDescription_Implementation() const
{
    return FText::Format(
        NSLOCTEXT("MyGame", "QuestProgressDesc", "Quest Progress: {0} +{1}"),
        FText::FromName(QuestID),
        FText::AsNumber(ProgressDelta));
}
```

**Design-Tipp:** `GetDisplayDescription` dynamisch mit den aktuellen Property-Werten befüllen. `"Quest Progress: KillDragon +1"` ist im Graph hundertmal hilfreicher als `"Quest Progress Update"`.
{% endhint %}

---

## Best Practices

* **Idempotenz anstreben.** Falls derselbe Dialog zweimal gespielt wird, sollte dein SideEffect nicht kaputt gehen. Ein Counter-Inkrement ist per se nicht idempotent — das ist OK, aber beachte es beim Design.
* **Kein User-Input in SideEffects.** Sie laufen inline und synchron. Kein Warten, keine Choice-Präsentation, keine async Aufrufe.
* **Kleine, fokussierte Klassen.** Lieber `SE_QuestProgress` und `SE_GrantAchievement` als ein Mega-SideEffect mit zehn Flags.
