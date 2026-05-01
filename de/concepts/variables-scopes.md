---
description: Dialogue-Scope vs. Participant-Scope — wann du welchen nimmst und wie du darauf zugreifst.
---

# Variablen & Scopes

MayDialogue hat zwei Orte, an denen Variablen leben. Die Wahl des richtigen Scopes ist die einzige Entscheidung, die du treffen musst — der Rest ist mechanisch gleich.

## Die zwei Scopes

| | Dialogue-Scope | Participant-Scope |
| --- | --- | --- |
| **Lebt** | Nur während des Gesprächs | Überdauert das Gespräch |
| **Gespeichert in** | Der laufenden Instanz | Der Participant-Komponente (SaveGame-markiert) |
| **Typischer Einsatz** | Zwischenstände, Counter, Aggregationen innerhalb einer Szene | „Hat mich schon getroffen", Beziehungswerte, Weltkenntnisse |

Faustregel: Wenn die Variable nach dem Dialog irrelevant ist → **Dialogue**. Wenn sie beim nächsten Gespräch noch gelten soll → **Participant**.

## Entscheidungsbaum

```text
Neue Variable gebraucht?
    │
    ├─ Nur für dieses Gespräch relevant?
    │       └─→ Dialogue-Scope
    │
    └─ Muss das Gespräch überdauern?
            ├─ Bis Level-Wechsel reicht?
            │       └─→ Participant-Scope (nicht persistiert)
            └─ Muss im SaveGame bleiben?
                    └─→ Participant-Scope (SaveGame automatisch)
```

**Beispiele:**

| Variable | Scope | Warum |
| --- | --- | --- |
| „Hat der Spieler schon nach dem Namen gefragt" | Dialogue | Nur diese Session relevant |
| „Wie oft wurde in dieser Szene provoziert" | Dialogue | Counter, Gespräch endet danach |
| „Spieler kennt das Geheimnis" | Participant + Save | Gilt für alle späteren Gespräche |
| „Friendship-Level mit NPC" | Participant + Save | Persistente Beziehung |
| „Hat in diesem Level schon gesprochen" | Participant, nicht persistiert | Gesprächsübergreifend, aber Level-scoped |

## Variablen deklarieren

Variablen werden im **Variables-Panel** des Asset-Editors angelegt — nicht im Blueprint, nicht im C++.

> 📸 **Bild-Platzhalter:** `variables-panel.png` — Variables-Panel mit mehreren Einträgen.
> *Setup:* Asset-Editor, Variables-Panel-Tab geöffnet. Tabelle mit vier Spalten: Name, Typ, Scope, Default. Drei Einträge sichtbar: 1. `HasAngered | Bool | Dialogue | false`. 2. `ProvocationCount | Int | Dialogue | 0`. 3. `HasMet | Bool | Participant | false`. Add-Button unten links sichtbar.

Unterstützte Typen: `Bool`, `Int`, `Float`, `String`, `Tag` (FGameplayTag).

## Variablen im Graph setzen

**SetVariable** gibt es in zwei Formen — gleiche Logik, andere visuelle Prominenz:

- Als **eigenständiger Action-Node** im Graph-Flow (wenn das Setzen der Hauptschritt ist).
- Als **SideEffect-Sub-Node** an einem anderen Node (wenn es nebenbei passiert).

> 📸 **Bild-Platzhalter:** `setvariable-as-node.png` — SetVariable-Node als eigenständige Box im Graph.
> *Setup:* Graph-Ausschnitt. `SayLine "Wer bist du?"` (Output-Pin) → `SetVariable`-Node (violette Title-Bar). Details-Panel rechts: `VariableName = HasAngered`, `Typ = Bool`, `Scope = Dialogue`, `Value = true`. Output-Pin des SetVariable-Nodes führt zu einem weiteren Node.

> 📸 **Bild-Platzhalter:** `setvariable-as-sideeffect.png` — SetVariable als SideEffect-Pill an einer SayLine.
> *Setup:* Eine SayLine im Graph ausgewählt. Im Node-Body unten: ein SideEffect-Pill mit Text `⚙ SetVariable: HasMet = true`. Kein eigenständiger Node im Graph — die Pill ist im Body der SayLine sichtbar.

## Variablen im Graph lesen

Lesen passiert über **Requirements** an Choice- oder Branch-Nodes:

- **CheckDialogueVariable** — prüft einen Wert im Dialogue-Scope.
- **CheckParticipantVariable** — prüft einen Wert im Participant-Scope.

> 📸 **Bild-Platzhalter:** `requirement-check-variable.png` — Branch-Node mit CheckDialogueVariable-Requirement.
> *Setup:* Branch-Node im Graph (blaue Title-Bar). Im Details-Panel rechts unter „Requirements": ein Eintrag `CheckDialogueVariable — ProvocationCount >= 3`. Output-Pins: True → SayLine „Genug davon!", False → SayLine (weiter wie gewohnt). Beide Pins sichtbar.

## Aus Blueprint lesen und schreiben

**Dialogue-Scope:**

> 📸 **Bild-Platzhalter:** `bp-dialogue-variable-read.png` — Blueprint: GetDialogueVariableBool auf der aktiven Instanz.
> *Setup:* BP-Graph eines EventHandlers. `Get Active Dialogue` (Subsystem) → `Get Dialogue Variable Bool`. Pin `Variable Name = "HasAngered"`. Return-Value führt zu einem Branch-Node. Alle Pins beschriftet.

```cpp
UMayDialogueInstance* Instance = UMayDialogueSubsystem::Get(this)->GetActiveDialogue();

// Schreiben
Instance->SetDialogueVariableBool("HasAngered", true);
Instance->SetDialogueVariableInt("ProvocationCount", 3);

// Lesen
bool Angered = Instance->GetDialogueVariableBool("HasAngered");
int32 Count  = Instance->GetDialogueVariableInt("ProvocationCount");
```

**Participant-Scope:**

```cpp
UMayDialogueParticipant* Part = Actor->FindComponentByClass<UMayDialogueParticipant>();

// Schreiben
Part->SetPersistentBool("HasMet", true);
Part->SetPersistentFloat("Friendship", 12.5f);

// Lesen mit Default
bool HasMet      = Part->GetPersistentBool("HasMet", false);
float Friendship = Part->GetPersistentFloat("Friendship", 0.0f);
```

## Auf Variablen-Änderungen reagieren

Jede Variablen-Mutation (beide Scopes) broadcastet `OnVariableChanged`. Dein Quest-System oder ein Analytics-Logger kann sich hier einklinken:

```cpp
Instance->OnVariableChanged.AddDynamic(this, &AQuestDirector::HandleVarChanged);

void AQuestDirector::HandleVarChanged(FName VarName, EMayDialogueVariableScope Scope,
                                       EMayDialogueVariableType Type, FString NewValue)
{
    if (VarName == "QuestAccepted" && NewValue == "true")
    {
        QuestSystem->StartQuest("FindTheKey");
    }
}
```

## Bekannte Einschränkungen

{% hint style="warning" %}
**SetVariable aus dem Graph schreibt aktuell nur in den Dialogue-Scope.** Für Participant-Scope aus dem Graph: Lege einen eigenen Blueprint-SideEffect an.

**Kurzanleitung:**

1. Content Browser → Rechtsklick → **Blueprint Class**, Parent Class: `UMayDialogueSideEffect`.
2. Die Funktion **Execute Side Effect** überschreiben (Event, nicht Pure).
3. Im Event-Body: aus dem `Context`-Pin **Get Instigator** oder **Get Target** aufrufen, um den gewünschten Actor zu holen.
4. Auf dem Actor **Get Component by Class → MayDialogueParticipant** aufrufen.
5. Auf der Participant-Komponente **Set Persistent Bool** (bzw. `Set Persistent Int`, `Set Persistent Float`, `Set Persistent String`, `Set Persistent Tag`) aufrufen.

Diesen SideEffect-Blueprint anschließend als Sub-Node an deiner SayLine oder deinem Action-Node hinzufügen.
{% endhint %}

{% hint style="info" %}
**Rechenoperationen** (z.B. `Friendship += 5`) sind im Graph nicht direkt möglich. Lösung: den aktuellen Wert in einem Blueprint-SideEffect auslesen, rechnen und zurückschreiben.
{% endhint %}

## Zusammenfassung

- **Dialogue-Scope**: nur während des Gesprächs, automatisch weg wenn der Dialog endet.
- **Participant-Scope**: überdauert Gespräche, SaveGame-ready.
- Typen: Bool, Int, Float, String, Tag.
- Deklaration im Variables-Panel des Asset-Editors.
- `OnVariableChanged` als Hook für externe Systeme.

Weiter: [Emotionen & Tags](emotions-tags.md).
