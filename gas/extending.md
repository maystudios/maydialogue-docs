---
description: Eigene GAS-Requirements und Aktions-Nodes per Blueprint — Schritt für Schritt.
---

# Eigene GAS-Nodes erstellen

Die drei eingebauten Requirements und vier SideEffects reichen nicht für jedes Projekt. Für projekt-spezifische GAS-Logik erstellst du eigene Subklassen — komplett in Blueprint, ohne C++.

---

## Eigenes GAS-Requirement (Blueprint)

Beispiel-Ziel: eine Choice erscheint nur, wenn eine bestimmte Quest abgeschlossen ist.

### Schritt 1 — Blueprint-Klasse anlegen

1. Content Browser → Rechtsklick → **Blueprint Class**.
2. Im Klassen-Picker **All Classes** öffnen.
3. `MayDialogueRequirement` eingeben und auswählen.
4. Klasse benennen, z.B. `BP_Req_QuestCompleted`.

> 📸 **Bild-Platzhalter:** `gasext-req-create-class.png` — "Pick Parent Class"-Dialog mit `MayDialogueRequirement` in der Suche.
> *Setup:* "All Classes"-Tab aktiv. Suchfeld enthält "MayDialogueRequirement". Ergebnis-Liste zeigt `UMayDialogueRequirement` mit vollständigem Pfad. Mauszeiger liegt auf dem Eintrag.

### Schritt 2 — Properties hinzufügen

Im Blueprint-Editor **Variables-Panel**: neue Variable anlegen.

* `QuestID` — Typ `FName`, **Instance Editable = true**, **Expose on Spawn = false**
* Optional: `bRequireCompleted` — Typ `bool`, Instance Editable = true

Instance Editable sorgt dafür, dass der Designer den Wert im Details-Panel des Nodes direkt eintragen kann.

> 📸 **Bild-Platzhalter:** `gasext-req-variables.png` — Variables-Panel des Blueprints mit `QuestID`-Eintrag.
> *Setup:* Blueprint-Editor offen, Variables-Panel links. `QuestID` vom Typ `Name` markiert. Rechts im Details-Panel: `Instance Editable = true` (Häkchen), `Category = "Quest"`.

### Schritt 3 — IsRequirementSatisfied überschreiben

Im Functions-Panel → **Override** → `Is Requirement Satisfied`.

Pseudo-Graph:

```text
Event Is Requirement Satisfied (Context)
  │
  ├─ Get Quest Subsystem (from Context → Get World)
  │
  ├─ Is Quest Completed? (QuestID)
  │     ├─ true  → Return Passed
  │     └─ false → Branch: bHideOnFail?
  │                   ├─ true  → Return FailedAndHidden
  │                   └─ false → Return FailedButVisible
```

{% hint style="info" %}
`bHideOnFail` ist bereits in der Basisklasse `UMayDialogueRequirement` definiert. Du musst ihn nicht selbst anlegen — er erscheint automatisch im Details-Panel jeder Subklasse.
{% endhint %}

> 📸 **Bild-Platzhalter:** `gasext-req-bp-graph.png` — Fertig implementierter `Is Requirement Satisfied`-Graph.
> *Setup:* Event `IsRequirementSatisfied` im Blueprint offen. Von links: `Event`-Node → `Get World` → `Get Quest Subsystem` → `Is Quest Completed (QuestID)` → Branch-Node. True-Pfad: `Return Passed`. False-Pfad: weiterer Branch auf `bHideOnFail` → zwei Returns (`FailedAndHidden` / `FailedButVisible`). Alle Verbindungen sichtbar.

### Schritt 4 — Testen

1. Blueprint kompilieren (Strg+F9).
2. Ein Dialog-Asset öffnen.
3. Auf eine Choice klicken → Details-Panel → **Add Requirement** → Dropdown → `BP_Req_QuestCompleted` auswählen.
4. `QuestID`-Feld ausfüllen.
5. Dialog kompilieren, im Preview-Runner testen.

---

## Eigener GAS-SideEffect (Blueprint)

Beispiel-Ziel: beim Betreten einer SayLine soll Quest-Progress um +1 erhöht werden.

### Schritt 1 — Blueprint-Klasse anlegen

1. Content Browser → Rechtsklick → **Blueprint Class**.
2. `MayDialogueSideEffect` als Parent wählen.
3. Benennen: `BP_SE_QuestProgressUpdate`.

> 📸 **Bild-Platzhalter:** `gasext-se-create-class.png` — "Pick Parent Class"-Dialog mit `MayDialogueSideEffect`.
> *Setup:* Wie bei Requirement-Schritt 1, aber `MayDialogueSideEffect` im Suchfeld. Ergebnis zeigt `UMayDialogueSideEffect`.

### Schritt 2 — Properties hinzufügen

* `QuestID` — FName, Instance Editable = true
* `ProgressDelta` — int32, Instance Editable = true, Standardwert = 1

### Schritt 3 — ExecuteSideEffect überschreiben

Functions-Panel → **Override** → `Execute Side Effect`.

Pseudo-Graph:

```text
Event Execute Side Effect (Context)
  │
  ├─ Get Quest Subsystem (from Context → Get World)
  │
  └─ Add Quest Progress (QuestID, ProgressDelta)
```

> 📸 **Bild-Platzhalter:** `gasext-se-bp-graph.png` — Execute-Side-Effect-Graph mit Quest-Progress-Aufruf.
> *Setup:* `Event ExecuteSideEffect` → `Get World` → `Get Quest Subsystem` → `Add Quest Progress` mit gepinnten `QuestID` und `ProgressDelta`-Variablen. Alle Pins verbunden, Blueprint kompiliert (kein Fehler-Symbol).

### Schritt 4 — Im Dialog verwenden

Auf einer SayLine im Details-Panel → **SideEffects** → **+** → `BP_SE_QuestProgressUpdate` → QuestID und ProgressDelta eintragen.

---

## Best Practices

* **Eine Sache pro Klasse.** Lieber `BP_Req_QuestCompleted` und `BP_Req_HasEnoughGold` als ein monolithisches "AllConditions"-Requirement.
* **Null-Guards einbauen.** Prüfe, ob dein Subsystem gefunden wurde — liefere bei fehlendem Subsystem `Passed` zurück, nicht `FailedAndHidden`. Sonst sind alle Choices weg, wenn dein Subsystem nicht lädt.
* **Display-Namen nutzen.** Trage in Class Settings einen sinnvollen **Blueprint Node Title** ein — das erscheint als Pill-Label im Graph.
* **Context immer über `GetWorld()` navigieren.** Nicht direkt auf `GWorld` zeigen.

---

{% hint style="success" %}
**Variante in C++**

Wenn du aus Performance- oder Typsicherheitsgründen in C++ arbeiten möchtest:

**Requirement:**
```cpp
UCLASS(BlueprintType, EditInlineNew, meta=(DisplayName="Quest Completed"))
class MYGAME_API UMyReq_QuestCompleted : public UMayDialogueRequirement
{
    GENERATED_BODY()
public:
    UPROPERTY(EditAnywhere, Category="Quest")
    FName QuestID;

    virtual EMayDialogueRequirementResult IsRequirementSatisfied_Implementation(
        const FMayDialogueContext& Context) const override;
};
```

```cpp
EMayDialogueRequirementResult UMyReq_QuestCompleted::IsRequirementSatisfied_Implementation(
    const FMayDialogueContext& Context) const
{
    if (QuestID == NAME_None) return EMayDialogueRequirementResult::Passed;

    auto* Quest = UQuestSubsystem::Get(Context.DialogueInstance->GetWorld());
    if (!Quest) return EMayDialogueRequirementResult::Passed;

    return Quest->IsCompleted(QuestID)
        ? EMayDialogueRequirementResult::Passed
        : (bHideOnFail
            ? EMayDialogueRequirementResult::FailedAndHidden
            : EMayDialogueRequirementResult::FailedButVisible);
}
```

**SideEffect:**
```cpp
UCLASS(BlueprintType, EditInlineNew, meta=(DisplayName="Quest Progress Update"))
class MYGAME_API UMySE_QuestProgressUpdate : public UMayDialogueSideEffect
{
    GENERATED_BODY()
public:
    UPROPERTY(EditAnywhere, Category="Quest") FName QuestID;
    UPROPERTY(EditAnywhere, Category="Quest") int32 ProgressDelta = 1;

    virtual void ExecuteSideEffect_Implementation(const FMayDialogueContext& Context) override;
};
```

```cpp
void UMySE_QuestProgressUpdate::ExecuteSideEffect_Implementation(const FMayDialogueContext& Context)
{
    if (QuestID == NAME_None) return;
    if (auto* Quest = UQuestSubsystem::Get(Context.DialogueInstance->GetWorld()))
    {
        Quest->AddProgress(QuestID, ProgressDelta);
    }
}
```

Beide Klassen erscheinen automatisch in den Node-Pickern, sobald das Projekt kompiliert ist.
{% endhint %}
