---
description: NPC erinnert sich beim zweiten Treffen – Participant-Variable + SaveGame-Persistenz.
---

# NPC erinnert sich beim 2. Treffen

## Szenario

Ein Informant reagiert beim ersten Treffen überrascht. Beim zweiten Gespräch – auch nach einem Spielstand-Ladevorgang – begrüßt er den Spieler mit Namen und erinnert sich an das Erste Treffen. Dafür kombinierst du eine persistente Bool-Variable mit einem Branch am Dialog-Einstieg.

## Was du lernst

- Persistente Bool-Variable im Participant anlegen.
- Beim ersten Besuch die Variable setzen (SideEffect am Exit-Node).
- Beim zweiten Besuch per Branch erkennen.
- Zusammenspiel mit dem SaveGame-System.

## Voraussetzungen

- [Beziehungs-Score tracken](relationship-counter.md) gelesen (Variablen-Konzept).
- [Persistence → SaveGame-Integration](../persistence/save-integration.md) verstanden.

## Mini-Graph

```text
[Entry]
   │
   ▼
[Branch]
   ├─ BP1: CheckVariable(HasMetBefore = true)
   │       → [SayLine: "Ah, du wieder! Was bringst du mir?"] → [PlayerChoice: ...]
   └─ BP2: <Fallback>
           → [SayLine: "Wer bist du? Ich kenne dich nicht."]
           → [SayLine: "...na gut. Ich heiße Corvus."]
           → [PlayerChoice: ...]

Alle Exits:
   └─ SideEffect am Exit-Node: SetVariable(HasMetBefore = true)
```

> 📸 **Bild-Platzhalter:** `npc-remembers-meeting-graph-overview.png` — Graph mit Branch für Erst-/Wiedertreffen und SideEffect am Exit.
> *Setup:* Asset `DA_Informant_Corvus` geöffnet. Entry → Branch (zwei Outputs). Oberer Pfad (Wiedertreffen): SayLine → PlayerChoice → Exit. Unterer Pfad (Ersttreffen): SayLine × 2 → PlayerChoice → Exit. Jeder Exit-Node hat SideEffect-Pill `SetVariable HasMetBefore = true` sichtbar.

## Schritt-für-Schritt

### 1. Persistente Variable anlegen

Asset: `DA_Informant_Corvus`. **Variables-Panel** → **Add Variable**:

| Property | Wert |
|----------|------|
| `Name` | `HasMetBefore` |
| `Type` | `Bool` |
| `Scope` | `Participant` |
| `DefaultValue` | `false` |
| `bPersistent` | `true` |

### 2. Branch für Erst-/Wiedertreffen

Branch-Node. BranchPoint[0]: **CheckVariable**:

| Property | Wert |
|----------|------|
| `VariableName` | `HasMetBefore` |
| `Scope` | `Participant` |
| `ComparisonOp` | `==` |
| `ComparisonValue` | `true` |

BranchPoint[1]: Fallback (kein Requirement).

### 3. Wiedertreffen-Pfad

BranchPoint[0]-Output → SayLine *„Ah, du wieder! Was bringst du mir?"* → PlayerChoice → Exit.

### 4. Ersttreffen-Pfad

BranchPoint[1]-Output → SayLine *„Wer bist du? Ich kenne dich nicht."* → SayLine *„...na gut. Ich heiße Corvus."* → PlayerChoice → Exit.

### 5. Variable beim Exit setzen

An **jedem Exit-Node** des Assets (es können mehrere sein) einen **SideEffect → SetVariable** anhängen:
- `Name: HasMetBefore`, `Scope: Participant`, `Value: true`.

So wird die Variable gesetzt, egal wie das Gespräch endet.

> 📸 **Bild-Platzhalter:** `npc-remembers-meeting-exit-sideffect.png` — Exit-Node mit SideEffect-Pill.
> *Setup:* Exit-Node ausgewählt. Im Node-Body: SideEffect-Pill `SetVariable: HasMetBefore = true`. Details-Panel zeigt die SideEffect-Properties.

### 6. SaveGame-Persistenz aktivieren

Auf der Participant-Komponente des NPC-Actors: `bPersistMemory = true`. MayDialogue schreibt `HasMetBefore` beim Dialog-Ende ins SaveGame und lädt es beim Spielstart.

### 7. Testen

PIE: Gespräch starten → Ersttreffen-Pfad läuft. Spiel speichern und laden. Gespräch erneut starten → Wiedertreffen-Pfad läuft.

## Blueprint-Triggering

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_Informant_Corvus
   └─ ...
```

> 📸 **Bild-Platzhalter:** `npc-remembers-meeting-bp-trigger.png` — Blueprint-Trigger am Informanten.
> *Setup:* Informanten-Actor BP. `Event OnInteract` → `Start Dialogue (MayDialogueLibrary)`. Pins: `Asset = DA_Informant_Corvus`, `Instigator = Get Player Pawn`, `Target = Self`.

## Variation / Weiter gehen

- Mehrere Treffenzähler: `MeetingCount` als Int, Increment bei jedem Exit. Ab Zahl 3 speziellen Pfad freischalten.
- Kombination mit Beziehungs-Score: [Beziehungs-Score tracken](relationship-counter.md).
- Variable von außen lesen: Quest-System prüft `HasMetBefore` vor dem Start und gibt ggf. einen Hinweis.

## Troubleshooting

**Wiedertreffen-Pfad kommt sofort beim ersten Spielstart.**
`DefaultValue = true` statt `false`. Variable-Panel prüfen.

**Variable setzt sich nach Spielstart zurück.**
`bPersistent = false` oder Participant-Komponente hat `bPersistMemory = false`. Beide Flags prüfen.

**SideEffect am Exit feuert nicht.**
SideEffect ist am falschen Node. Es gibt mehrere Exit-Nodes im Asset und nicht alle haben den SideEffect. Jeden Exit prüfen.
