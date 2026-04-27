---
description: Tags prüfen, Effekte anwenden, Variablen zählen – der vollständige GAS-Loop in einem Dialog.
---

# GAS-getriebener Dialog

## Szenario

Ein Kräuterweib bietet dem Spieler einen Heiltrank an. Sie prüft seinen Gesundheitszustand (Attribut), erkennt ob er sie schon kennt (Tag), heilt ihn auf Wunsch (GameplayEffect), merkt sich die Trankkäufe (Participant-Variable) und vergibt beim ersten Treffen einen Story-Tag. Dieses Rezept zeigt den kompletten GAS-Loop: lesen, schreiben, reagieren.

## Was du lernst

- Branch auf HasTag für Ersttreffen vs. Wiedertreffen.
- CheckAttribute-Requirement an einer Choice.
- ApplyEffect-Action-Node für einen Heiltrank.
- SideEffect-Sub-Node für einen Zähler-Increment.
- AddTag dauerhaft via GAS-Variante setzen.

## Voraussetzungen

- [Verzweigungen mit Bedingungen](branching-conditions.md) abgeschlossen.
- GAS aktiv, AttributeSet mit `Health` registriert.

## Mini-Graph

```text
[Entry]
   │
   ▼
[Branch]
   ├─ BP1: HasTag(Story.Met.Herbwoman) → [SayLine: "Schön, dich wiederzusehen."]
   └─ BP2: <Fallback>                  → [SayLine: "Ich kenne dich noch nicht."]
                                               │
                                          [AddTag: Story.Met.Herbwoman → Player]
                                               │
                                        ┌──────┴──────────────────────────┐
                                        ▼
                                  [PlayerChoice]
                                    ├─ "Einen Trank bitte."  (Req: Health < 50, FailedButVisible)
                                    │       └─► [ApplyEffect: GE_SmallHeal]
                                    │               ├─ SideEffect: PotionsBought += 1
                                    │               └─► [SayLine: "Besser?"] → [Exit: Completed]
                                    └─ "Nein danke." → [Exit: Completed]
```

> 📸 **Bild-Platzhalter:** `gas-driven-dialogue-graph-overview.png` — Vollständiger Kräuterweib-Dialog im Asset-Editor.
> *Setup:* Asset `DA_Herbwoman_Visit` geöffnet. Sichtbar: Entry → Branch (Diamant, zwei Outputs). Oberer Pfad: SayLine Wiedertreffen → PlayerChoice. Unterer Pfad: SayLine Ersttreffen → AddTag-Node (hellgrüne Box) → PlayerChoice. PlayerChoice mit zwei Choices, obere Choice hat Lock-Icon (Requirement). Von Wahl 1: ApplyEffect-Node (lila Box) → SayLine → Exit. Von Wahl 2: direkt Exit.

## Schritt-für-Schritt

### 1. Tags und Variable definieren

In `DefaultGameplayTags.ini`:
```ini
+GameplayTagList=(Tag="Story.Met.Herbwoman",DevComment="Spieler kennt die Kräuterfrau")
+GameplayTagList=(Tag="Dialogue.Speaker.Herbwoman",DevComment="")
```

Im Asset `DA_Herbwoman_Visit` unter **Variables-Panel**: Variable `PotionsBought`, Typ `Int`, Scope `Participant`, Default `0`.

### 2. Branch für Erst-/Wiedertreffen

Branch-Node einfügen. BranchPoint[0]: HasTag-Requirement:

| Property | Wert |
|----------|------|
| `RequiredTag` | `Story.Met.Herbwoman` |
| `bCheckOnInstigator` | `true` |

BranchPoint[1]: leer (Fallback).

### 3. Ersttreffen-Pfad: Tag setzen

Nach der einführenden SayLine ein **AddTag**-Action-Node:

| Property | Wert |
|----------|------|
| `TargetParticipantTag` | `Dialogue.Participant.Player` |
| `Tag` | `Story.Met.Herbwoman` |
| `bApplyPermanent` | `true` (via Infinite-GE) |

> 📸 **Bild-Platzhalter:** `gas-driven-dialogue-addtag-details.png` — Details-Panel des AddTag-Nodes.
> *Setup:* AddTag-Node ausgewählt. Details: `TargetParticipantTag = Dialogue.Participant.Player`, `Tag = Story.Met.Herbwoman`, `bApplyPermanent = true`.

### 4. PlayerChoice mit Attribut-Gate

Erster Choice-Eintrag bekommt ein **CheckAttribute**-Requirement:

| Property | Wert |
|----------|------|
| `Attribute` | `UVHSAttributeSet::Health` |
| `ComparisonOp` | `<` |
| `ComparisonValue` | `50.0` |
| `FailureResult` | `FailedButVisible` |
| `UnavailableReason` | *„Du scheinst gesund."* |

Der Spieler sieht die Option immer – kann sie aber nur wählen wenn Health < 50. Bei Hover erscheint die Begründung.

### 5. ApplyEffect mit Zähler-SideEffect

Vom Heilungs-Choice-Output → **ApplyEffect**-Node:

| Property | Wert |
|----------|------|
| `EffectClass` | `GE_SmallHeal` |
| `TargetParticipantTag` | `Dialogue.Participant.Player` |
| `EffectLevel` | `1.0` |

Am ApplyEffect-Node **SideEffect → SetVariable**:
- `Name: PotionsBought`, `Scope: Participant`, `Op: Increment`, `Value: 1`.

> 📸 **Bild-Platzhalter:** `gas-driven-dialogue-applyeffect-details.png` — ApplyEffect-Node mit SideEffect-Pill.
> *Setup:* ApplyEffect-Node ausgewählt. Details: `EffectClass = GE_SmallHeal`, `TargetParticipantTag = Dialogue.Participant.Player`. Im Node-Body unten: SideEffect-Pill `SetVariable PotionsBought += 1`.

### 6. Compile und PIE-Test

Im Debugger nach dem Trank prüfen: Health steigt, `Story.Met.Herbwoman` liegt am Player-ASC, `PotionsBought = 1`.

## Loose-Tag vs. GameplayEffect

| Modus | Pro | Contra |
|-------|-----|--------|
| `LooseGameplayTag` | Einfach, kein GE nötig | Nicht repliziert, nicht persistiert |
| Via Infinite-GE | Repliziert, persistiert | Braucht einen GE-Asset |

Für Story-Tags die ins SaveGame sollen: **immer GE-Variante**.

## Blueprint-Triggering

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset:      DA_Herbwoman_Visit
   ├─ Instigator: Get Player Pawn
   └─ Target:     Self
```

## Variation / Weiter gehen

- `PotionsBought` persistent speichern → [Persistence → Participant-Memory](../persistence/participant-memory.md).
- CheckAttribute-Requirement für eine ganze Branch-Entscheidung statt nur eine Choice nutzen.
- Eigenes GAS-Requirement bauen (z.B. „Spieler hat Ability X") → [Eigenen Requirement in Blueprint bauen](custom-blueprint-requirement.md).

## Troubleshooting

**ApplyEffect zeigt keine Wirkung.**
`EffectClass` ist leer. Target hat keinen ASC. Der GE ist Instant, aber das AttributeSet klemmt `Current = Max` nicht in `PostGameplayEffectExecute`.

**Tag gesetzt, Branch nimmt trotzdem Fallback.**
Loose-Tag und Branch prüfen im selben Frame. In seltenen Fällen cached MayDialogue den Tag-State – Workaround: GE-Variante nutzen.

**CheckAttribute liefert nie Passed.**
Attribut im Dropdown leer oder falsch verlinkt. Das AttributeSet muss am ASC des Spielers registriert sein.
