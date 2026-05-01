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

Den `Context`-Eingang öffnest du via Helper-Knoten aus der `MayDialogueLibrary` (siehe [Custom Requirements](custom-requirements.md)) oder direkt — alle drei Context-Felder sind seit v1.0 `BlueprintReadOnly`.

Pseudo-Graph:

```text
Event Execute Node (Context)
  │
  ├─ Get World From Context  →  Get Quest Subsystem
  │
  ├─ Report Quest Step Reached (QuestStepName)
  │
  ├─ Execute Side Effects (Context)    ← wichtig: SideEffects der Sub-Nodes ausführen
  │
  └─ Return: [Make Advance] (Get First Valid Output)
```

`Execute Side Effects` und `Get First Valid Output` sind Blueprint-Callable-Methoden von `UMayDialogueNode_Base` — du rufst sie auf `Self` auf. Für den Return-Wert nutze die **Make-\***-Nodes aus `UMayDialogueLibrary` (Kategorie `MayDialogue|Task Result`):

| Make-Node | Wann |
| --- | --- |
| `Make Advance` | Normales Weiterschalten zum nächsten Node |
| `Make Abort` | Dialog abbrechen |
| `Make Wait` | Dialog pausieren (Async-Nodes) |
| `Make Pause And Present Choices` | Choice-Screen öffnen |
| `Make Advance With Choice` | Choice + direktes Advance |
| `Make Return To Start` | An den Anfang des Graphen zurückspringen |
| `Make Return To Last` | Zum letzten besuchten Node zurück |
| `Make Return To Current` | Aktuellen Node wiederholen |

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

Wenn dein Node auf ein äußeres Event warten soll (z.B. eine Animation, einen Timer, ein Netzwerk-Ergebnis), gibt er `Make Wait` zurück und löst später den Advance aus. **Das funktioniert jetzt vollständig in Blueprint** via `UMayDialogueAsyncLibrary`:

```text
Event Execute Node (Context)
  │
  ├─ Merke NextNodeGuid = Get First Valid Output (Self, Context)
  │
  ├─ Starte Timer / binde Event
  │
  └─ Return: [Make Wait]

[Beim Timer-Ablauf / Event-Trigger]
  │
  └─ Request Node Advance (Context.DialogueInstance, NextNodeGuid)
       ↑ aus UMayDialogueAsyncLibrary — BP-Callable, null-guarded
```

`Request Node Advance` (Kategorie `MayDialogue|Async`) ist ein dünner BP-Wrapper um `Instance->ForceTransitionToNode`. Er fügt einen Null-Guard hinzu und ist über den Discovery-Namen in der Node-Palette leicht auffindbar.

Für Multiplayer-Projekte: wenn du mehrere SideEffects auf einem Async-Node kombinieren willst, nutze `Execute All Side Effects` und `Execute All Client Side Effects` aus `UMayDialogueAsyncLibrary`, um die Client/Server-Aufteilung korrekt zu halten.

---

## Variante in C++

Setup-Grundlagen (Build.cs, BlueprintNativeEvent-Pattern, Module-Reload, Editor-Sichtbarkeit) → [C++-Erweiterung — Grundlagen](cpp-fundamentals.md).

### Header (`MyDN_NotifyQuest.h`)

```cpp
#pragma once

#include "CoreMinimal.h"
#include "Nodes/MayDialogueNode_Base.h"
#include "MyDN_NotifyQuest.generated.h"

struct FMayDialogueContext;
struct FMayDialogueTaskResult;

UCLASS(BlueprintType, meta=(DisplayName="Notify Quest"))
class MYGAME_API UMyDN_NotifyQuest : public UMayDialogueNode_Base
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Quest")
    FName QuestStepName;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Quest")
    bool bMarkCompleted = false;

    virtual FMayDialogueTaskResult ExecuteNode_Implementation(
        const FMayDialogueContext& Context) override;

    virtual FText GetNodeDisplayName_Implementation() const override;
    virtual FLinearColor GetNodeColor_Implementation() const override;
};
```

### Implementation (`MyDN_NotifyQuest.cpp`)

```cpp
#include "MyDN_NotifyQuest.h"
#include "Instance/MayDialogueInstance.h"
#include "Types/MayDialogueTypes.h"
#include "MyGame/Quest/QuestSubsystem.h"

FMayDialogueTaskResult UMyDN_NotifyQuest::ExecuteNode_Implementation(
    const FMayDialogueContext& Context)
{
    if (QuestStepName != NAME_None)
    {
        UWorld* World = Context.DialogueInstance ? Context.DialogueInstance->GetWorld() : nullptr;
        if (UQuestSubsystem* Quest = World ? UQuestSubsystem::Get(World) : nullptr)
        {
            if (bMarkCompleted) Quest->MarkCompleted(QuestStepName);
            else                Quest->ReportStepReached(QuestStepName);
        }
    }

    // Sub-Node-SideEffects ausführen — die Basisklasse macht das NICHT automatisch,
    // wenn du ExecuteNode überschreibst. Sonst würden Designer-konfigurierte
    // SideEffects auf dem Node ignoriert.
    ExecuteSideEffects(Context);

    // FailBehavior wird in der Engine über die Pre-Execution-Requirements-Evaluation
    // gehandhabt — du gibst hier nur den Erfolgs-Pfad zurück.
    // FMayDialogueTaskResult::Advance / Abort / Wait / PauseAndPresentChoices etc.
    // sind die C++-Static-Factories. In Blueprint sind diese als Make Advance / Make Abort /
    // Make Wait / Make Pause And Present Choices etc. in UMayDialogueLibrary verfügbar.
    return FMayDialogueTaskResult::Advance(GetFirstValidOutput(Context));
}

FText UMyDN_NotifyQuest::GetNodeDisplayName_Implementation() const
{
    if (QuestStepName == NAME_None)
    {
        return NSLOCTEXT("MyGame", "NotifyQuestEmpty", "Notify Quest (no step)");
    }
    return FText::Format(
        NSLOCTEXT("MyGame", "NotifyQuestFmt", "Notify Quest: {0}{1}"),
        FText::FromName(QuestStepName),
        bMarkCompleted ? FText::FromString(TEXT(" ✓")) : FText::GetEmpty());
}

FLinearColor UMyDN_NotifyQuest::GetNodeColor_Implementation() const
{
    return FLinearColor(0.2f, 0.45f, 0.8f);   // Quest-blue, klar abgrenzbar von SayLine/Branch
}
```

{% hint style="info" %}
**`BlueprintNativeEvent` — kein UFUNCTION-Override im Subclass.** Override `ExecuteNode_Implementation`, NICHT `ExecuteNode`. Wenn du es trotzdem mit `UFUNCTION(...)` markierst, gibt UHT einen Fehler. `GetNodeDisplayName` und `GetNodeColor` folgen demselben Pattern.
{% endhint %}

{% hint style="warning" %}
**`ExecuteSideEffects(Context)` nicht vergessen.** Wenn du `ExecuteNode_Implementation` überschreibst, übernimmst du die volle Kontrolle — die Basisklasse ruft die SideEffects-Sub-Nodes nicht mehr für dich auf. Vergiss diesen Aufruf NICHT, sonst werden Designer-konfigurierte SideEffects (Add Tag, Apply Effect, deine eigenen) ignoriert.
{% endhint %}

### Async Custom-Node (Wait-on-Event-Pattern)

Wenn dein Node auf etwas wartet (Animation, Timer, externe API), gib `Wait()` zurück und löse später per `UMayDialogueAsyncLibrary::RequestNodeAdvance(...)` aus:

```cpp
FMayDialogueTaskResult UMyDN_AwaitWebhook::ExecuteNode_Implementation(
    const FMayDialogueContext& Context)
{
    UMayDialogueInstance* Instance = Context.DialogueInstance;
    if (!Instance) return FMayDialogueTaskResult::Abort();

    const FGuid NextNode = GetFirstValidOutput(Context);

    // Async-State auf der Instance registrieren — siehe MayDialogueNodeAsyncState.h
    UMyDN_AwaitWebhookState* State = Instance->GetOrCreateAsyncState<UMyDN_AwaitWebhookState>(NodeGuid);
    State->NextNodeGuid = NextNode;
    State->BindCompletionDelegate([Instance, NextNode]()
    {
        // UMayDialogueAsyncLibrary::RequestNodeAdvance ist der öffentliche
        // BP-callable Wrapper (Kategorie "MayDialogue|Async"). Er enthält
        // einen Null-Guard und ist der empfohlene Weg auch aus C++.
        UMayDialogueAsyncLibrary::RequestNodeAdvance(Instance, NextNode);
    });

    return FMayDialogueTaskResult::Wait();
}
```

Vergiss in `CleanupAsyncState` die Delegate-Unbinds nicht — sonst leakt dein Node nach Dialog-Abort. Siehe `MayDialogueNode_Wait.cpp` und `MayDialogueNode_PlayAnimation.cpp` als Referenz-Implementations im Plugin.

### C++ + Blueprint mischen

Wenn du einen reusable Custom-Node-Typ baust, machs als C++-Basis (`Blueprintable`) mit der Subsystem-Bridge in C++, und lass Designer dann konkrete BP-Subclasses ableiten (`BP_DN_QuestStart`, `BP_DN_QuestComplete`, …). Spart Build-Cycles bei Iteration.

Sobald dein C++-Modul kompiliert ist, erscheint der Node genau wie ein BP-Node im Kontext-Menü unter der `Class Settings → Category` — Editor-Reload genügt, kein Asset-Refresh.

---

## Anmerkungen

* **Graph-Darstellung** kommt automatisch — dein Node wird als Standard-Task-Node gerendert (blaue Box). Für eine Spezialdarstellung wäre eine Slate-Node-Factory nötig (C++, fortgeschritten).
* **Kategorisierung** macht den Unterschied. Ohne Kategorie landet der Node unter "Misc" — vergrabene Nodes werden nicht genutzt.
* **Mehrere Ausgänge** sind möglich: trage mehrere GUIDs in `OutputNodeGuids` ein. Designer verbindet dann mehrere Ausgangs-Pins.
