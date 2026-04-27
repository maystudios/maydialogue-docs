---
description: Eigene Requirements per Blueprint — am Beispiel "Hat Quest abgeschlossen".
---

# Eigene Requirements

Requirements steuern, ob ein Choice, Branch oder SayLine aktiv ist. Du kannst eigene Requirements für jede projekt-spezifische Bedingung erstellen: Quest-Status, Inventar, Tageszeit, Reputationswert.

Beispiel-Ziel: eine Choice erscheint nur, wenn der Spieler eine bestimmte Quest abgeschlossen hat — `BP_Req_QuestCompleted`.

---

## Schritt 1 — Blueprint-Klasse anlegen

1. Content Browser → Rechtsklick → **Blueprint Class**.
2. **All Classes** öffnen.
3. `MayDialogueRequirement` suchen und auswählen.
4. Benennen: `BP_Req_QuestCompleted`.

> 📸 **Bild-Platzhalter:** `creq-step1-picker.png` — "Pick Parent Class"-Dialog mit `MayDialogueRequirement`.
> *Setup:* Blueprint-Klassen-Dialog, "All Classes"-Tab. Suchfeld: "MayDialogueRequirement". Ergebnis: `UMayDialogueRequirement` mit Modul-Pfad sichtbar. Select-Button aktiv.

---

## Schritt 2 — Properties anlegen

Variables-Panel → neue Variable:

* `QuestID` — Typ `FName`, **Instance Editable = true**, Category = "Quest"
* Optional: `RequiredStage` — Typ `int32`, Instance Editable = true, Standardwert = 0, Category = "Quest"

> 📸 **Bild-Platzhalter:** `creq-step2-variables.png` — Variables-Panel mit `QuestID` und `RequiredStage`.
> *Setup:* Blueprint-Editor. Variables-Panel zeigt `QuestID` (Name, hervorgehoben) und `RequiredStage` (Integer, Standardwert 0). Details-Panel rechts: `Instance Editable = true`, `Category = Quest`.

---

## Schritt 3 — IsRequirementSatisfied überschreiben

Functions-Panel → **Override** → `Is Requirement Satisfied`.

Pseudo-Graph:

```text
Event Is Requirement Satisfied (Context)
  │
  ├─ Is QuestID valid? (None-Check)
  │     └─ true → Return Passed  (keine Quest = immer bestanden)
  │
  ├─ Get Quest Subsystem (Context → DialogueInstance → Get World)
  │
  ├─ Is Quest Completed? (QuestID, RequiredStage)
  │     ├─ true  → Return Passed
  │     └─ false → Branch: bHideOnFail?
  │                   ├─ true  → Return FailedAndHidden
  │                   └─ false → Return FailedButVisible
```

> 📸 **Bild-Platzhalter:** `creq-step3-graph.png` — Vollständiger Is-Requirement-Satisfied-Graph.
> *Setup:* `Event Is Requirement Satisfied (Context)` → `Is Valid QuestID` (None-Vergleich) → Branch. True-Pfad direkt: `Return Passed`. False-Pfad: `Get World → Get Quest Subsystem → Is Quest Completed (QuestID)` → Branch. True: `Return Passed`. False: `Branch: bHideOnFail` → zwei Return-Nodes (`FailedAndHidden` / `FailedButVisible`). Alle Pfade verbunden, kein offener Exec-Pin.

{% hint style="info" %}
`bHideOnFail` ist in der Basisklasse `UMayDialogueRequirement` enthalten und muss nicht selbst deklariert werden. Er erscheint im Details-Panel und steuert, ob `FailedAndHidden` oder `FailedButVisible` zurückgegeben wird.
{% endhint %}

---

## Schritt 4 — Im Dialog verwenden

1. Dialog-Asset öffnen.
2. Eine Choice auswählen → Details-Panel → **Add Requirement** → `BP_Req_QuestCompleted`.
3. `QuestID` eintragen (z.B. `KillDragon`).
4. `bHideOnFail` je nach Design auf true oder false.
5. Kompilieren und im Preview-Runner testen.

> 📸 **Bild-Platzhalter:** `creq-step4-details.png` — Details-Panel einer Choice mit `BP_Req_QuestCompleted` als Sub-Node.
> *Setup:* MayDialogue-Editor, Choice "Ich habe den Drachen getötet" ausgewählt. Requirements-Array im Details-Panel zeigt einen Eintrag: `BP_Req_QuestCompleted`. Darunter: `QuestID = KillDragon`, `RequiredStage = 0`, `bHideOnFail = true`. Felder gut lesbar, Pill-Label im Graph zeigt "QuestCompleted: KillDragon".

---

## Composable Requirements — mehrere stapeln

Auf derselben Choice können mehrere Requirements liegen:

```text
Choice "Ich habe alles vorbereitet"
  Requirements:
    1. Quest Completed: KillDragon          bHideOnFail: true
    2. Quest Completed: CollectIngredients  bHideOnFail: true
    3. Check GAS Attribute: Gold >= 100     FailResult: FailedButVisible
```

Ergebnis: Choice erscheint nur wenn beide Quests erledigt sind. Ist Gold zu wenig: Choice sichtbar, aber deaktiviert (mit Tooltip).

> 📸 **Bild-Platzhalter:** `creq-stacked-requirements.png` — Details-Panel einer Choice mit drei gestapelten Requirements.
> *Setup:* Choice-Node im MayDialogue-Editor ausgewählt. Requirements-Array zeigt drei Einträge: `BP_Req_QuestCompleted (KillDragon, bHideOnFail=true)`, `BP_Req_QuestCompleted (CollectIngredients, bHideOnFail=true)`, `UMayDlgRequirement_CheckAttribute (Gold >= 100, FailResult=FailedButVisible)`. Alle drei sichtbar, Array-Elemente nummeriert.

---

## Typische projekt-spezifische Requirements

| Requirement | Prüft |
| --- | --- |
| `Req_QuestActive(QuestID)` | Quest läuft gerade |
| `Req_QuestCompleted(QuestID)` | Quest abgeschlossen (Beispiel oben) |
| `Req_InventoryContains(ItemID, Count)` | Spieler hat Item |
| `Req_TimeOfDay(MinHour, MaxHour)` | Tageszeit im Bereich |
| `Req_AreaVisited(AreaTag)` | Spieler war in diesem Bereich |
| `Req_NPCReputation(NPCTag, MinLevel)` | Reputationswert hoch genug |

Jedes davon ist eine kleine Blueprint-Klasse, die Designer direkt aus dem Dropdown-Menü auswählt.

---

{% hint style="success" %}
**Variante in C++**

```cpp
UCLASS(BlueprintType, EditInlineNew, meta=(DisplayName="Quest Completed"))
class MYGAME_API UMyReq_QuestCompleted : public UMayDialogueRequirement
{
    GENERATED_BODY()
public:
    UPROPERTY(EditAnywhere, Category="Quest")
    FName QuestID;

    UPROPERTY(EditAnywhere, Category="Quest")
    int32 RequiredStage = 0;

    virtual EMayDialogueRequirementResult IsRequirementSatisfied_Implementation(
        const FMayDialogueContext& Context) const override;

    virtual FText GetDisplayDescription_Implementation() const override;
};
```

```cpp
EMayDialogueRequirementResult UMyReq_QuestCompleted::IsRequirementSatisfied_Implementation(
    const FMayDialogueContext& Context) const
{
    if (QuestID == NAME_None) return EMayDialogueRequirementResult::Passed;

    auto* Quest = UQuestSubsystem::Get(Context.DialogueInstance->GetWorld());
    if (!Quest) return EMayDialogueRequirementResult::Passed;

    return Quest->IsCompletedAtStage(QuestID, RequiredStage)
        ? EMayDialogueRequirementResult::Passed
        : (bHideOnFail
            ? EMayDialogueRequirementResult::FailedAndHidden
            : EMayDialogueRequirementResult::FailedButVisible);
}

FText UMyReq_QuestCompleted::GetDisplayDescription_Implementation() const
{
    return FText::Format(
        NSLOCTEXT("MyGame", "QuestCompletedDesc", "Quest abgeschlossen: {0}"),
        FText::FromName(QuestID));
}
```

`GetDisplayDescription` liefert den Text, der als Pill-Label im Graph erscheint. Eine gute Beschreibung macht Designer-Workflows drastisch einfacher.
{% endhint %}
