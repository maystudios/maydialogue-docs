# Eigene Nodes per Blueprint

Manchmal reichen die 19 vordefinierten Nodes nicht. Z.B. *„NotifyQuestSystem"* oder *„PlayLevelSequence"*. Du kannst eigene Node-Typen per Blueprint erstellen.

## Schritt 1 – Blueprint-Klasse anlegen

1. Content Browser → Rechtsklick → **Blueprint Class**.
2. In der Class-Suche **All Classes** öffnen.
3. `MayDialogueNode_Base` wählen.
4. Benennen: `BP_DN_NotifyQuest`.

## Schritt 2 – Event-Graph bearbeiten

Öffne den Blueprint. Gehe zum **Functions**-Panel.

**Override Function**: `Execute Node`. Implementierung:

```
Event Execute Node (Context):
  - Get Quest Subsystem from Context's World
  - Call Quest Subsystem::ReportDialogReached(QuestStepName)
  - Return FMayDialogueTaskResult::Advance(First Valid Output)
```

### Hilfsmethode: Get First Valid Output

```
Func GetFirstValidOutput(Context):
  return UMayDialogueNode_Base::GetFirstValidOutput(Self, Context)
```

Das gibt automatisch die nächste GUID aus dem `OutputNodeGuids`-Array, deren Requirements (falls vorhanden) passed.

## Schritt 3 – Properties hinzufügen

Im Blueprint-Editor: **Variables → Add Variable**. Typisch:

* `QuestStepName: FName` (Expose in Details-Panel, Instance Editable = true).

## Schritt 4 – Kategorie & Display-Name

In den **Class Settings** (oben rechts im Blueprint-Editor):

* **Category**: `"MayDialogue|Actions|MyGame"` — damit der Node im Kontext-Menü unter einer eigenen Kategorie erscheint.

## Schritt 5 – Testen

1. Dialog-Asset öffnen.
2. Rechtsklick → Kontext-Menü zeigt deinen Node (unter `MyGame → NotifyQuest`).
3. Platzieren, Property befüllen, verbinden.

## Advanced: Async Nodes

Wenn dein Node auf externes warten soll:

1. Registriere dich als Async-Node:

   ```
   Context.DialogueInstance::RegisterActiveAsyncNode(Self)
   ```

2. Merke dir `NextNodeGuid`.

3. Starte deine Wait-Logik (Timer, Event, …).

4. Beim Trigger:

   ```
   Context.DialogueInstance::ForceTransitionToNode(NextNodeGuid)
   ```

5. Returne aus `ExecuteNode` **kein** direktes Advance, sondern pausiere die Ausführung. Der TaskResult-Typ `PauseDialogueAndPresentChoices` ist semantisch für Choices; für generisches Warten gibt es noch keine eigene Enum-Variante — aktuell wird das am Instance-Niveau gehandhabt.

## C++-Variante

Wenn du stattdessen in C++ arbeiten willst:

```cpp
UCLASS(meta=(DisplayName="Notify Quest"))
class MYGAME_API UMyDN_NotifyQuest : public UMayDialogueNode_Base
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, Category="Quest")
    FName QuestStepName;

    virtual FMayDialogueTaskResult ExecuteNode_Implementation(const FMayDialogueContext& Context) override;
};
```

```cpp
FMayDialogueTaskResult UMyDN_NotifyQuest::ExecuteNode_Implementation(const FMayDialogueContext& Context)
{
    if (auto* Quest = UQuestSubsystem::Get(Context.GetWorld()))
    {
        Quest->ReportDialogReached(QuestStepName);
    }

    // SideEffects ausführen
    ExecuteSideEffects(Context);

    // Zum nächsten Node
    return FMayDialogueTaskResult::Advance(GetFirstValidOutput(Context));
}
```

## Anmerkungen

* **Graph-Visuelles** kommt automatisch. Dein Node wird als Standard-Task-Node gerendert. Wenn du eine Spezial-Darstellung brauchst: Slate-Graph-Node-Factory erweitern (C++, fortgeschritten).
* **Kategorisierung** im Kontext-Menü macht Auffindbarkeit drastisch besser.
* **Vergiss nicht `SideEffects` auszuführen**, wenn dein Node Sub-Node-Arrays haben soll.
