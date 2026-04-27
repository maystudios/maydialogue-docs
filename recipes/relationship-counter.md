---
description: Beziehungs-Score persistent im Participant tracken und im Dialog darauf reagieren.
---

# Beziehungs-Score tracken

## Szenario

Du willst, dass ein wichtiger NPC sich merkt, wie oft und wie der Spieler mit ihm gesprochen hat. Ein `Relationship`-Int-Wert steigt bei freundlichen Antworten und sinkt bei unhöflichen. Ab Wert 10 gibt der NPC eine Quest frei. Dieser Score überlebt Spielstände, weil er in der Participant-Memory liegt.

## Was du lernst

- Participant-Variable anlegen (Scope: Participant, Int).
- SetVariable-SideEffect für Increment/Decrement konfigurieren.
- CheckVariable-Requirement nutzen, um auf den Score zu reagieren.
- Participant-Memory persistent speichern.

## Voraussetzungen

- [GAS-getriebener Dialog](gas-driven-dialogue.md) abgeschlossen.
- [Persistence → Participant-Memory](../persistence/participant-memory.md) gelesen.

## Mini-Graph

```text
[Entry]
   │
   ▼
[Branch]
   ├─ BP1: CheckVariable(Relationship >= 10) → [SayLine: "Ich vertraue dir jetzt. Hier die Quest."] → [Exit: QuestGranted]
   └─ BP2: <Fallback>                         → [SayLine: "Wie kann ich dir helfen?"] → [PlayerChoice]

[PlayerChoice]
   ├─ "Freundlich antworten."  → SideEffect: Relationship += 2   → [Exit]
   ├─ "Neutral antworten."     → SideEffect: Relationship += 1   → [Exit]
   └─ "Unhöflich antworten."   → SideEffect: Relationship -= 1   → [Exit]
```

> 📸 **Bild-Platzhalter:** `relationship-counter-graph-overview.png` — Vollständiger Relationship-Dialog im Asset-Editor.
> *Setup:* Asset `DA_Advisor_Talk` geöffnet. Entry → Branch (zwei Outputs). Oberer Pfad: SayLine "Quest frei" → Exit. Unterer Pfad: SayLine → PlayerChoice mit drei Choices, jede mit SideEffect-Pill am Choice-Sub-Node sichtbar (Relationship += 2 / += 1 / -= 1).

## Schritt-für-Schritt

### 1. Participant-Variable anlegen

Asset: `DA_Advisor_Talk`. **Variables-Panel** öffnen → **Add Variable**:

| Property | Wert |
|----------|------|
| `Name` | `Relationship` |
| `Type` | `Int` |
| `Scope` | `Participant` |
| `DefaultValue` | `0` |
| `bPersistent` | `true` |

`Scope: Participant` = die Variable lebt auf der Participant-Komponente des NPCs, nicht nur für die Dauer des Dialogs.

> 📸 **Bild-Platzhalter:** `relationship-counter-variables-panel.png` — Variables-Panel mit der Relationship-Variable.
> *Setup:* Variables-Panel Tab geöffnet. Einziger Eintrag: `Relationship | Int | Participant | Default: 0 | Persistent: true`.

### 2. Branch mit CheckVariable

Branch-Node einfügen. BranchPoint[0]: **Requirements → Add → CheckVariable**:

| Property | Wert |
|----------|------|
| `VariableName` | `Relationship` |
| `Scope` | `Participant` |
| `ComparisonOp` | `>=` |
| `ComparisonValue` | `10` |

BranchPoint[1]: leer (Fallback).

### 3. PlayerChoice mit Relationship-SideEffects

PlayerChoice-Node anlegen. Drei Choices:

**Choice 1** (Freundlich): Am Choice-Sub-Node **SideEffect → SetVariable**:
- `Name: Relationship`, `Op: Increment`, `Value: 2`.

**Choice 2** (Neutral): SideEffect mit `Op: Increment`, `Value: 1`.

**Choice 3** (Unhöflich): SideEffect mit `Op: Decrement`, `Value: 1`.

Alle drei Choices → Exit.

### 4. Persistenz aktivieren

Die Participant-Komponente des NPCs muss `bPersistMemory = true` haben. MayDialogue schreibt die Variable dann beim Dialogue-End in das SaveGame. Details siehe [Participant-Memory](../persistence/participant-memory.md).

### 5. Compile und testen

Im PIE: Gespräch dreimal mit freundlicher Antwort → Relationship = 6. Noch zweimal → Relationship = 10 → beim nächsten Öffnen kommt der Quest-Pfad.

## Relationship-Wert von außen lesen

```text
[Get Participant Variable (Int)]
   ├─ Participant:    NPC Actor Component
   ├─ VariableName:   "Relationship"
   └─► Return Value → Use in Quest System
```

```cpp
int32 Score = NPC->GetParticipantComponent()->GetVariableInt(TEXT("Relationship"));
```

> 📸 **Bild-Platzhalter:** `relationship-counter-bp-read.png` — Blueprint-Graph zum Lesen des Relationship-Werts.
> *Setup:* Blueprint eines Quest-Systems. `Get Participant Component (NPC)` → `Get Variable Int ("Relationship")` → `Branch (>= 10)` → Quest-Grant-Logik.

## Variation / Weiter gehen

- Mehrere Relationship-Variablen für verschiedene Fraktionen: `Rel_Guards`, `Rel_Merchants`, `Rel_Rebels`.
- Relationship-Score im HUD anzeigen: über das [Read/Write-API](../runtime/read-write-api.md) von außen abfragen.
- Negative Schwelle: bei Relationship <= -5 → NPC verweigert weitere Gespräche (`FailedAndHidden` auf alle Choices).

## Troubleshooting

**Variable setzt sich bei jedem Spielstart zurück.**
`bPersistent = false` oder die Participant-Komponente hat `bPersistMemory = false`. Beides prüfen.

**Branch nimmt nie den Quest-Pfad, obwohl Score >= 10.**
`Scope` im CheckVariable-Requirement ist `Dialogue` statt `Participant`. Dialogue-Scope existiert nur für die Laufzeit des Dialogs und ist nach jedem Start 0.

**SideEffect-Increment passiert nicht sichtbar.**
SideEffects feuern, wenn der Node *betreten* wird. Bei Choices feuert der SideEffect, wenn die Choice *ausgewählt* und ihr Output-Node betreten wird – nicht beim Anzeigen der Choice-Liste.
