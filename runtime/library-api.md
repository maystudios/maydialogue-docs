---
description: UMayDialogueLibrary — statische Blueprint-Helfer für den schnellen Einstieg.
---

# Blueprint-Library

`UMayDialogueLibrary` ist eine Blueprint Function Library mit statischen Methoden. Sie ist der einfachste Weg, MayDialogue aus jedem beliebigen Blueprint anzusteuern — ohne vorher eine Subsystem-Referenz zu cachen.

{% hint style="info" %}
Die Library ist ein reiner Convenience-Layer. Sie hat keinen eigenen State und delegiert alles ans Subsystem. Wenn du Delegates binden willst, musst du das Subsystem direkt ansprechen — die Library hat keine eigenen Events.
{% endhint %}

---

## Methoden im Überblick

### Basis-Library (`UMayDialogueLibrary`)

| Funktion | Art | Wann nutzen |
|---|---|---|
| `Start Dialogue` | Callable | Dialog mit bestimmtem Asset starten. Liefert Instance oder `nullptr`. |
| `Stop Dialogue` | Callable | Eine bestimmte Instance abbrechen. |
| `Stop All Dialogues` | Callable | Alle aktiven Dialoge abbrechen (Spielertod, Pause, Level-Wechsel). |
| `Get Active Dialogue` | Pure | Aktive Instance lesen (z.B. um Variablen abzufragen). |
| `Is Any Dialogue Active` | Pure | Prüfen ob gerade ein Dialog läuft. |
| `Get Dialogue Subsystem` | Callable | Subsystem-Referenz holen (z.B. für Delegate-Binding). |
| `Get All Participants` | Pure | Alle aktiven `UMayDialogueParticipant`s in der Welt. |
| `Find Participant By Tag` | Pure | Participant anhand eines `FGameplayTag` suchen. |

#### TaskResult Make-Helpers (Kategorie `MayDialogue|Task Result`)

Für den Return-Wert von `ExecuteNode`-Overrides in Custom-Node-Blueprints:

| Make-Node | Bedeutung |
|---|---|
| `Make Advance` | Normales Weiterschalten (mit optionalem NextNodeGuid-Override) |
| `Make Abort` | Dialog abbrechen |
| `Make Wait` | Dialog pausieren (Async-Nodes) |
| `Make Pause And Present Choices` | Choice-Screen öffnen |
| `Make Advance With Choice` | Choice + direktes Advance |
| `Make Return To Start` | An den Anfang des Graphen zurückspringen |
| `Make Return To Last` | Zum letzten besuchten Node zurück |
| `Make Return To Current` | Aktuellen Node wiederholen |

---

### GAS-Library (Kategorie `MayDialogue|GAS`)

| Funktion | Art | Beschreibung |
|---|---|---|
| `Is Dialogue Server Authoritative` | Pure | Ob der aktuelle Dialogue-Context server-autoritativ ist |
| `Get ASC From Context` | Pure | `UAbilitySystemComponent*` aus `FMayDialogueContext` — kein manueller Cast |
| `Trigger Cue On Context` | Callable | Gameplay Cue direkt aus einem Context feuern |

---

### Async-Library (Kategorie `MayDialogue|Async`)

| Funktion | Art | Beschreibung |
|---|---|---|
| `Request Node Advance` | Callable | Async-Node-Transition auslösen — BP-Wrapper um `ForceTransitionToNode` mit Null-Guard |
| `Evaluate All Requirements` | Pure | Alle Requirements eines Arrays evaluieren und aggregiertes Ergebnis liefern |
| `Execute All Side Effects` | Callable | Server-seitige SideEffects eines Arrays ausführen |
| `Execute All Client Side Effects` | Callable | Client-seitige (kosmetische) SideEffects eines Arrays ausführen |

---

### Variables-Library (Kategorie `MayDialogue|Variables`)

| Funktion | Art | Beschreibung |
|---|---|---|
| `Get Dialogue Variable` | Callable | Variable aus einem `FMayDialogueVariables`-Container lesen |
| `Set Dialogue Variable` | Callable | Variable in einen `FMayDialogueVariables`-Container schreiben |
| `Copy Dialogue Variables` | Callable | **Geplant** — noch nicht verfügbar. |

---

## Wann Library, wann Subsystem direkt?

| Szenario | Empfehlung |
|---|---|
| Schneller Start-Aufruf aus einem Widget oder LevelScript | Library |
| Du brauchst sowieso das Subsystem (Event-Binding) | Subsystem direkt und Referenz cachen |
| Wiederholte Calls im selben Scope | Subsystem-Referenz einmal holen und cachen |

---

## Rezepte

### Dialog starten

> 📸 **Bild-Platzhalter:** `library-start-dialogue-bp.png` — BP-Graph: Start Dialogue Library-Call mit Fehler-Branch.
> *Setup:* Interaction-Component-Blueprint oder LevelScript. `Event On Interact` → `Start Dialogue` (Library, Kategorie MayDialogue). Pins: `Asset` = Dialog-Asset-Referenz, `Instigator` = Player-Pawn-Referenz, `Target` = Self. Return-Value-Pin → `Is Valid` → Branch: True = (weiter), False = `Print String "Dialog failed"`.

```text
[Start Dialogue]  (Kategorie: MayDialogue)
  ├─ Asset:      DA_VillagerIntro
  ├─ Instigator: Player Pawn
  └─ Target:     Self (NPC)
       │ Return Value
       ▼
[Is Valid]
  ├─ True  → Dialog läuft
  └─ False → Fehler-Fallback
```

---

### Laufenden Dialog lesen

> 📸 **Bild-Platzhalter:** `library-get-active-bp.png` — BP-Graph: Is Any Dialogue Active → Branch → Get Active Dialogue.
> *Setup:* HUD-Blueprint, `Event Tick` oder Button-Press-Event. `Is Any Dialogue Active` → Branch. True-Zweig: `Get Active Dialogue` → (UMayDialogueInstance-Referenz weiterverarbeiten, z.B. `Get Status`).

```text
[Is Any Dialogue Active]
    │ (bool)
    ▼
[Branch]
  └─ True → [Get Active Dialogue] → Instance benutzen
```

---

### Alle Dialoge beim Spielertod abbrechen

> 📸 **Bild-Platzhalter:** `library-stop-all-bp.png` — BP-Graph: On Player Died → Stop All Dialogues.
> *Setup:* Character-Blueprint oder GameMode. `Event On Player Died` (Custom Event) → `Stop All Dialogues` (Library-Node). Keine weiteren Verbindungen nötig.

```text
[Event On Player Died]
    │
    ▼
[Stop All Dialogues]
```

---

### Subsystem holen für Delegate-Binding

```text
[Get Dialogue Subsystem]
    │ (Subsystem-Referenz in Variable speichern)
    ▼
[Bind Event to On Any Dialogue Started]  ──► Custom Event: Handle Dialogue Started
[Bind Event to On Any Dialogue Ended]    ──► Custom Event: Handle Dialogue Ended
```

---

## C++-Nutzung

```cpp
// Dialog starten
UMayDialogueInstance* Inst = UMayDialogueLibrary::StartDialogue(
    this, DA_VillagerIntro, Player, NPC);

// Prüfen ob Dialog läuft
if (UMayDialogueLibrary::IsAnyDialogueActive(this))
{
    UMayDialogueLibrary::StopAllDialogues(this);
}

// Subsystem holen
UMayDialogueSubsystem* Sub = UMayDialogueLibrary::GetDialogueSubsystem(this);
```

{% hint style="info" %}
In C++ ist der direkte Subsystem-Zugriff idiomatischer als die Library. Die Library ist primär für Blueprint-Nutzer gedacht.
{% endhint %}
