---
description: Eigene Dialog-Nodes per Blueprint — Schritt für Schritt.
---

# Eigene Nodes

Die vordefinierten Nodes (SayLine, PlayerChoice, Branch, …) reichen nicht für jedes Szenario. Wenn dein Graph z.B. das Quest-System benachrichtigen oder eine Level-Sequenz starten soll, erstellst du einen eigenen Node-Typ.

Beispiel-Ziel: ein Node `BP_DN_NotifyQuest`, der beim Durchlaufen einen Quest-Schritt als "erreicht" meldet.

---

## Schritt 1 — Blueprint-Klasse anlegen

1. Content Browser → Rechtsklick → **Blueprint Class**.
2. Im Klassen-Picker **All Classes** öffnen.
3. `MayDialogueNode_Base` eintippen und auswählen.
4. Benennen: `BP_DN_NotifyQuest`.

> 📸 **Bild-Platzhalter:** `cnode-step1-picker.png` — "Pick Parent Class"-Dialog, `MayDialogueNode_Base` in der Suche ausgewählt.
> *Setup:* Blueprint-Klassen-Dialog. "All Classes"-Tab aktiv. Suchfeld: "MayDialogueNode_Base". Ergebnis-Liste zeigt die Klasse mit vollständigem Modul-Pfad. Mauszeiger auf dem Eintrag, Select-Button hervorgehoben.

---

## Schritt 2 — Properties hinzufügen

Im Variables-Panel:

* `QuestStepName` — Typ `FName`, **Instance Editable = true**, Category = "Quest"
* Optional: `bMarkCompleted` — Typ `bool`, Instance Editable = true, Category = "Quest"

> 📸 **Bild-Platzhalter:** `cnode-step2-variables.png` — Variables-Panel mit `QuestStepName` und `bMarkCompleted`.
> *Setup:* Blueprint-Editor offen. Variables-Panel links zeigt `QuestStepName` (Typ Name, hervorgehoben) und `bMarkCompleted` (Typ Bool). Rechts im Details-Panel: `Instance Editable = true` (Häkchen), `Category = Quest`.

---

## Schritt 3 — ExecuteNode überschreiben

Im Functions-Panel → **Override** → `Execute Node`.

Pseudo-Graph:

```text
Event Execute Node (Context)
  │
  ├─ Get World from Context (Context → DialogueInstance → Get World)
  │
  ├─ Get Quest Subsystem (World)
  │
  ├─ Report Quest Step Reached (QuestStepName)
  │
  ├─ Execute Side Effects (Context)    ← wichtig: SideEffects der Sub-Nodes ausführen
  │
  └─ Return: Advance to First Valid Output
```

`Execute Side Effects` und `Get First Valid Output` sind Blueprint-Callable-Methoden von `UMayDialogueNode_Base` — du rufst sie auf `Self` auf.

> 📸 **Bild-Platzhalter:** `cnode-step3-graph.png` — Vollständiger Execute-Node-Graph mit Quest-Aufruf und Return.
> *Setup:* Blueprint-Editor, `ExecuteNode`-Funktion offen. Von links: `Event Execute Node (Context)` → `Get World` (aus Context.DialogueInstance) → `Get Quest Subsystem` → `Report Step Reached (QuestStepName-Variable)` → `Execute Side Effects (Context, Self)` → `Get First Valid Output (Context, Self)` → `Return Node` mit dem Rückgabewert. Alle Verbindungen sichtbar, keine offenen Pins.

{% hint style="warning" %}
**`Execute Side Effects` nicht vergessen.** Wenn dein Node SideEffect-Sub-Nodes unterstützen soll (und das sollte er), musst du `ExecuteSideEffects(Context)` explizit aufrufen. Die Basisklasse macht das nicht automatisch für dich.
{% endhint %}

---

## Schritt 4 — Kategorie und Display-Name setzen

In **Class Settings** (oben rechts im Blueprint-Editor):

* **Blueprint Node Title**: `Notify Quest`
* **Category**: `MayDialogue|Actions|MyGame`

> 📸 **Bild-Platzhalter:** `cnode-step4-classsettings.png` — Class-Settings-Panel mit Node-Title und Category-Eintrag.
> *Setup:* Blueprint-Editor, Class-Settings-Tab (Zahnrad-Symbol oben). Felder "Blueprint Display Name" und "Category" sichtbar. Werte: Name = "Notify Quest", Category = "MayDialogue|Actions|MyGame". Umgebung zeigt übrige Class-Settings-Felder grau/leer.

---

## Schritt 5 — Im Dialog verwenden

1. Dialog-Asset öffnen.
2. Im Graphen Rechtsklick → Kontext-Menü → Kategorie `MyGame` → **Notify Quest** wählen.
3. Node platzieren, `QuestStepName` eintragen.
4. Eingangs-Pin vom vorherigen Node verbinden, Ausgangs-Pin mit dem nächsten Node verbinden.
5. Dialog kompilieren (Strg+F9), im Preview-Runner testen.

> 📸 **Bild-Platzhalter:** `cnode-step5-in-graph.png` — Fertig verdrahteter Graph: SayLine → NotifyQuest-Node → Exit.
> *Setup:* MayDialogue-Editor, Asset `DA_QuestDialogue`. Von links: `SayLine "Aufgabe erledigt."` (dunkelrot) → `Notify Quest`-Node (blauer Task-Node, QuestStepName = "KillDragon_Deliver") → `Exit Completed`-Node (rot). Alle Verbindungen horizontal, Node-Title "Notify Quest" sichtbar.

---

## Async-Nodes (fortgeschritten)

Wenn dein Node auf ein äußeres Event warten soll (z.B. eine Animation, einen Timer, ein Netzwerk-Ergebnis):

```text
Event Execute Node (Context)
  │
  ├─ Register as Async Node (Context.DialogueInstance, Self)
  │
  ├─ Merke NextNodeGuid = GetFirstValidOutput(Context)
  │
  ├─ Starte Timer / binde Event
  │
  └─ Return: (kein direktes Advance — Dialog wartet)

[Beim Timer-Ablauf / Event-Trigger]
  │
  └─ Force Transition To Node (Context.DialogueInstance, NextNodeGuid)
```

---

{% hint style="success" %}
**Variante in C++**

```cpp
UCLASS(meta=(DisplayName="Notify Quest"))
class MYGAME_API UMyDN_NotifyQuest : public UMayDialogueNode_Base
{
    GENERATED_BODY()
public:
    UPROPERTY(EditAnywhere, Category="Quest")
    FName QuestStepName;

    virtual FMayDialogueTaskResult ExecuteNode_Implementation(
        const FMayDialogueContext& Context) override;
};
```

```cpp
FMayDialogueTaskResult UMyDN_NotifyQuest::ExecuteNode_Implementation(
    const FMayDialogueContext& Context)
{
    if (auto* Quest = UQuestSubsystem::Get(Context.DialogueInstance->GetWorld()))
    {
        Quest->ReportDialogReached(QuestStepName);
    }

    ExecuteSideEffects(Context);
    return FMayDialogueTaskResult::Advance(GetFirstValidOutput(Context));
}
```

Sobald das Projekt kompiliert ist, erscheint der C++-Node genau wie ein Blueprint-Node im Kontext-Menü.
{% endhint %}

---

## Anmerkungen

* **Graph-Darstellung** kommt automatisch — dein Node wird als Standard-Task-Node gerendert (blaue Box). Für eine Spezialdarstellung wäre eine Slate-Node-Factory nötig (C++, fortgeschritten).
* **Kategorisierung** macht den Unterschied. Ohne Kategorie landet der Node unter "Misc" — vergrabene Nodes werden nicht genutzt.
* **Mehrere Ausgänge** sind möglich: trage mehrere GUIDs in `OutputNodeGuids` ein. Designer verbindet dann mehrere Ausgangs-Pins.
