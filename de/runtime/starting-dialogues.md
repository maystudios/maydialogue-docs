---
description: Drei Methoden, einen Dialog zu starten — mit Blueprint und C++.
---

# Einen Dialog starten

Alle drei Wege münden in dieselbe Instance. Such dir den aus, der am besten in deinen Code passt.

---

## Variante 1 — Participant-Komponente

**Wann:** Der NPC hat eine `MayDialogueParticipant`-Komponente und soll seinen `DefaultDialogue` starten, sobald der Spieler interagiert.

### Blueprint

> 📸 **Bild-Platzhalter:** `start-participant-bp.png` — BP-Graph des NPC-Interaktions-Events.
> *Setup:* NPC-Blueprint öffnen. Graph zeigt: `Event On Interact` (Custom Event, ein `AActor`-Pin `Instigator`) → `Get Component by Class` (Class: `MayDialogueParticipant`, Target: Self) → `Start Default Dialogue` (Instigator-Pin: `Instigator`-Variable). Rückgabe-Pin (UMayDialogueInstance) bleibt unverbunden. Alle Nodes sauber beschriftet.

```text
[Event On Interact (Instigator: Actor)]
    │
    ▼
[Get Component by Class: MayDialogueParticipant] (Target: Self)
    │
    ▼
[Start Default Dialogue]
    └─ Instigator: Instigator-Pin von On Interact
```

### C++

```cpp
void AGuardActor::OnInteract(AActor* Instigator)
{
    if (auto* Part = FindComponentByClass<UMayDialogueParticipant>())
    {
        Part->StartDefaultDialogue(Instigator);
    }
}
```

{% hint style="info" %}
`StartDefaultDialogue` startet den im `DefaultDialogue`-Feld der Komponente eingetragenen Asset. Willst du einen anderen Asset spielen, setze vorher `SetActiveDialogue(MeinAsset)`.
{% endhint %}

---

## Variante 2 — Library (empfohlen für Blueprints)

**Wann:** Du willst einen Dialog mit einem bestimmten Asset starten, ohne vorher Referenzen auf Participant-Komponenten zu haben. Funktioniert aus jedem Blueprint: Widget, GameMode, LevelScript.

### Blueprint

> 📸 **Bild-Platzhalter:** `start-library-bp.png` — BP-Graph mit Library-Node.
> *Setup:* Beliebiges Blueprint (z.B. LevelBlueprint). Graph: `Event BeginPlay` → `Start Dialogue` (Library-Node aus Kategorie MayDialogue). Pins befüllt: `Asset` = hartcodierter Dialog-Asset-Verweis, `Instigator` = `Get Player Pawn (0)`, `Target` = Referenz auf NPC-Actor aus Scene. Return-Value-Pin geht in einen `Is Valid`-Branch.

```text
[Start Dialogue]  (Kategorie: MayDialogue)
  ├─ Asset:      DA_Greeting
  ├─ Instigator: Get Player Pawn (0)
  └─ Target:     Guard Actor Referenz
       │
       ▼
[Is Valid: Return Value]
  ├─ True  → Dialog läuft
  └─ False → Print String: "Dialog konnte nicht starten"
```

### C++

```cpp
UMayDialogueInstance* Inst = UMayDialogueLibrary::StartDialogue(
    this,           // WorldContext (beliebiges UObject in der Welt)
    DA_Greeting,    // UMayDialogueAsset*
    PlayerPawn,     // Instigator
    GuardActor      // Target
);
```

---

## Variante 3 — Subsystem direkt

**Wann:** Dein System-Code (Quest-Director, Cutscene-Manager, Tutorial-Script) hat das Subsystem ohnehin referenziert und braucht auch die Delegates.

### Blueprint

> 📸 **Bild-Platzhalter:** `start-subsystem-bp.png` — BP-Graph mit Subsystem-Zugriff.
> *Setup:* Blueprint mit `Get MayDialogue Subsystem` → `Start Dialogue` (am Subsystem-Node). `Instigator` = Player Pawn, `Target` = NPC-Referenz. Subsystem-Referenz in einer lokalen Variable gecacht, dann darunter direkt `Bind Event to On Any Dialogue Ended` mit einem Custom-Event verbunden.

```text
[Get MayDialogue Subsystem]
    │
    ├─→ [Start Dialogue]
    │       ├─ Asset:      DA_IntroScene
    │       ├─ Instigator: Player Pawn
    │       └─ Target:     NPC Ref
    │
    └─→ [Bind Event to On Any Dialogue Ended]
            └─ Event: Handle Dialogue Ended (Custom Event)
```

### C++

```cpp
if (auto* Sub = GetWorld()->GetSubsystem<UMayDialogueSubsystem>())
{
    Sub->OnAnyDialogueEnded.AddDynamic(this, &AQuestDirector::HandleDialogueEnded);
    Sub->StartDialogue(DA_IntroScene, PlayerPawn, NPC);
}
```

---

## Vorher prüfen: CanStartDialogue

Willst du wissen, ob ein Dialog starten kann, ohne ihn zu starten:

```text
[Get MayDialogue Subsystem] → [Can Start Dialogue]
  ├─ Asset:      DA_Greeting
  ├─ Instigator: Player Pawn
  └─ Target:     Guard Actor
       │ (bool)
       ▼
[Branch]
  ├─ True  → Interaktions-Prompt anzeigen
  └─ False → Prompt ausblenden
```

```cpp
bool bOk = Sub->CanStartDialogue(DA_Greeting, Player, Guard);
```

Prüft: Asset gültig, Entry vorhanden, mindestens ein Participant mit passenden Tags.

---

## Laufenden Dialog abbrechen

```text
[Get MayDialogue Subsystem] → [Stop All Dialogues]
```

```cpp
Sub->StopAllDialogues();   // alle abbrechen (Level-Wechsel, Spielertod)
Sub->StopDialogue(Inst);   // gezielt eine Instance abbrechen
```

> 📸 **Bild-Platzhalter:** `stop-all-dialogues-bp.png` — BP-Graph für Spielertod-Handling.
> *Setup:* Game Mode oder Character-Blueprint. `Event On Player Died` → `Stop All Dialogues` (Library-Node). Kein weiterer Output.

{% hint style="warning" %}
Wenn du `StartDialogue` rufst, während bereits ein Dialog läuft, wird der alte **sofort abgebrochen**. Kein Queue, kein Crash.
{% endhint %}
