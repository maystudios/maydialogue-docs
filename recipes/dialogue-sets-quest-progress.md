---
description: Dialog setzt Quest-Tags und feuert Events – so treibt ein Gespräch den Quest-Fortschritt voran.
---

# Dialog setzt Quest-Progress

## Szenario

Ein Gespräch mit einem Zeugen soll den Quest-Schritt *„Zeugen befragt"* abschließen. Das passiert über einen AddTag-Action-Node und einen FireEvent-Node am Ende des Dialogs. Das Quest-System hört auf den Event und aktualisiert seinen Zustand. Kein externes Scripting nötig – alles passiert innerhalb des Dialogs.

## Was du lernst

- AddTag-Action-Node für Quest-Progress-Tags nutzen.
- FireEvent-Node, um externe Systeme zu benachrichtigen.
- SideEffect an Exit-Node als Alternative zu Action-Nodes.
- Wann Action-Node vs. SideEffect für Quest-Aktionen.

## Voraussetzungen

- [Quest-Status im Dialog lesen](quest-status-in-dialogue.md) abgeschlossen.
- Quest-System vorhanden, das auf GameplayTag-Events hört.

## Mini-Graph

```text
[Entry]
   │
   ▼
[SayLine: "Ich habe den Verdächtigen gestern Nacht gesehen."]
   │
   ▼
[PlayerChoice]
   ├─ "Wann genau?" → [SayLine: "Gegen Mitternacht."]
   └─ "Wer war dabei?" → [SayLine: "Er war allein."]
         │
         └──────────────────────────────┐
                                        ▼
                               [AddTag: Quest.WitnessInterviewed → Player]
                                        │
                                        ▼
                               [FireEvent: Quest.Event.WitnessInterviewed]
                                        │
                                        ▼
                               [Exit: Completed]
```

> 📸 **Bild-Platzhalter:** `dialogue-sets-quest-progress-graph-overview.png` — Dialog-Graph mit AddTag und FireEvent-Nodes.
> *Setup:* Asset `DA_Witness_Talk` geöffnet. SayLine → PlayerChoice mit zwei Choices → beide Pfade münden in dieselbe Node-Kette: AddTag-Node (hellgrüne Box) → FireEvent-Node (orange Box) → Exit. Layout zeigt die Zusammenführung der Pfade deutlich.

## Schritt-für-Schritt

### 1. Quest-Tags und Event-Tags definieren

In `DefaultGameplayTags.ini`:
```ini
+GameplayTagList=(Tag="Quest.WitnessInterviewed")
+GameplayTagList=(Tag="Quest.Event.WitnessInterviewed")
```

### 2. Dialog-Graph aufbauen

Asset: `DA_Witness_Talk`. Speaker: `Dialogue.Speaker.Witness`. Normaler Dialog-Flow mit SayLines und PlayerChoice.

### 3. AddTag-Node für Quest-Progress

Am Ende beider Choice-Pfade münden die Verbindungen in einen **AddTag**-Action-Node:

| Property | Wert |
|----------|------|
| `TargetParticipantTag` | `Dialogue.Participant.Player` |
| `Tag` | `Quest.WitnessInterviewed` |
| `bApplyPermanent` | `true` |

> 📸 **Bild-Platzhalter:** `dialogue-sets-quest-progress-addtag-details.png` — Details-Panel des AddTag-Nodes.
> *Setup:* AddTag-Node ausgewählt. Details: `TargetParticipantTag = Dialogue.Participant.Player`, `Tag = Quest.WitnessInterviewed`, `bApplyPermanent = true`.

### 4. FireEvent-Node für externe Systeme

Vom AddTag-Output → **FireEvent**-Node:

| Property | Wert |
|----------|------|
| `EventTag` | `Quest.Event.WitnessInterviewed` |
| `Payload` | *(optional, leer lassen)* |

Der FireEvent-Node broadcastet das Event über das MayDialogue-Event-System. Externe Subscriber (Quest-System, Achievement-System) empfangen es via `OnDialogueEvent`-Delegate.

### 5. Quest-System subscriben

```cpp
// Im Quest-System, beim Start:
UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(GetWorld());
Sub->OnDialogueEvent.AddUObject(this, &UQuestSystem::HandleDialogueEvent);

// Handler:
void UQuestSystem::HandleDialogueEvent(FGameplayTag EventTag, UMayDialogueInstance* Inst)
{
    if (EventTag == TAG_Quest_Event_WitnessInterviewed)
    {
        AdvanceQuestStep(EQuestStep::WitnessInterviewed);
    }
}
```

### 6. Compile und PIE-Test

Im PIE: Dialog komplett durchführen → im GAS-Debugger `Quest.WitnessInterviewed` am Player-ASC prüfen. Quest-System-Log auf den Event-Empfang prüfen.

## Action-Node vs. SideEffect – wann was?

| Situation | Empfehlung |
|-----------|------------|
| Quest-Progress ist der Hauptpunkt (prominent im Graph) | AddTag **Action-Node** + FireEvent **Action-Node** |
| Tag nebenbei gesetzt, wenig visuell | **SideEffect** am Exit-Node |
| Mehrere Schritte in Folge | Action-Nodes für Lesbarkeit |

Für Quest-Progress empfehlen sich Action-Nodes – so sieht man im Graph auf einen Blick, was das Gespräch bewirkt.

## Blueprint-Alternative: OnDialogueEnded

Falls du kein FireEvent willst, kannst du auch auf den Dialog-Abschluss in Blueprint reagieren:

```text
[On Dialogue Ended (Delegate)]
   ├─ Check: ExitStatus == Completed
   └─► [Quest System → Advance Step: WitnessInterviewed]
```

> 📸 **Bild-Platzhalter:** `dialogue-sets-quest-progress-bp-handler.png` — Blueprint-Handler für OnDialogueEnded.
> *Setup:* Quest-Manager-Blueprint. `Bind Event to On Dialogue Ended` → `Switch on Exit Status` → `Completed` Branch → `Advance Quest Step`. Participant-Referenz über `Get Instigator` aus dem Instance-Handle.

## Variation / Weiter gehen

- FireEvent mit **Payload** nutzen: mitgeben, welcher Zeuge befragt wurde (Tag als Payload).
- **Wait-Node** der auf ein Quest-Event wartet: Dialog wartet auf externe Bestätigung → [Wait auf GameplayEvent](wait-for-event.md).
- Quest-Status vor dem Dialog prüfen → [Quest-Status im Dialog lesen](quest-status-in-dialogue.md).

## Troubleshooting

**Tag gesetzt, aber Quest-System reagiert nicht.**
Subscription auf `OnDialogueEvent` fehlt oder zu spät gebunden. Prüfe die Bind-Reihenfolge – Subsystem muss vor dem Dialog-Start subscribed sein.

**FireEvent feuert, aber EventTag stimmt nicht.**
`EventTag` im FireEvent-Node und in der `HandleDialogueEvent`-Abfrage unterschiedlich. Tag-Namen exakt vergleichen.

**AddTag im Dedicated-Server nicht repliziert.**
AddTag greift auf den lokalen ASC. In Singleplayer kein Problem. Für Server-authoritative Setups: Tag via RPC auf dem Server setzen lassen.
