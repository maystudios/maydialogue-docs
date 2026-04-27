---
description: Quest-Status per GameplayTag im Dialog abfragen und verschiedene Gesprächspfade ausliefern.
---

# Quest-Status im Dialog lesen

## Szenario

Ein Questgeber reagiert unterschiedlich, je nachdem ob der Spieler die zugehörige Quest noch nicht hat, sie aktiv hat oder sie abgeschlossen hat. Die Quest-Status werden als GameplayTags am Spieler-ASC geführt. Der Dialog-Branch liest diese Tags und wählt den passenden Pfad.

## Was du lernst

- Mehrere HasTag-Requirements an einem Branch für drei Zustände konfigurieren.
- Priorisierung der BranchPoints für korrekte Reihenfolge nutzen.
- Quest-Tags am Spieler-ASC lesen (Tag-Konvention).

## Voraussetzungen

- [Verzweigungen mit Bedingungen](branching-conditions.md) abgeschlossen.
- Quest-System setzt Tags: `Quest.FindArtifact.Completed`, `Quest.FindArtifact.Active`.

## Mini-Graph

```text
[Entry]
   │
   ▼
[Branch]
   ├─ BP1: HasTag(Quest.FindArtifact.Completed)
   │       → [SayLine: "Exzellent! Du hast das Artefakt! Hier deine Belohnung."] → [ApplyEffect: GE_QuestReward] → [Exit: Completed]
   ├─ BP2: HasTag(Quest.FindArtifact.Active)
   │       → [SayLine: "Du bist noch dran. Das Artefakt liegt südlich vom Turm."] → [Exit]
   └─ BP3: <Fallback>
           → [SayLine: "Ich brauche deine Hilfe. Willst du einen Auftrag?"] → [PlayerChoice: Annehmen/Ablehnen]
```

> 📸 **Bild-Platzhalter:** `quest-status-in-dialogue-graph-overview.png` — Asset-Editor mit Branch und drei BranchPoints.
> *Setup:* Asset `DA_QuestGiver_Artifact` geöffnet. Entry → Branch-Node (Diamant, drei Ausgangs-Pins). Oberer Pin → kurze Kette SayLine + ApplyEffect → Exit. Mittlerer Pin → SayLine → Exit. Unterer Pin → SayLine → PlayerChoice → zwei Exits. Alle drei Pfade gut lesbar auseinandergezogen.

## Schritt-für-Schritt

### 1. Quest-Tags definieren

In `DefaultGameplayTags.ini`:
```ini
+GameplayTagList=(Tag="Quest.FindArtifact.Active")
+GameplayTagList=(Tag="Quest.FindArtifact.Completed")
```

### 2. Branch mit drei BranchPoints

Branch-Node einfügen. Drei BranchPoints in dieser Reihenfolge:

**BranchPoint[0]:** HasTag `Quest.FindArtifact.Completed`, `bCheckOnInstigator: true`.
**BranchPoint[1]:** HasTag `Quest.FindArtifact.Active`, `bCheckOnInstigator: true`.
**BranchPoint[2]:** Kein Requirement (Fallback – Quest noch nicht gestartet).

Die Reihenfolge ist entscheidend: `Completed` muss vor `Active` geprüft werden, da ein Spieler nach Abschluss möglicherweise beide Tags trägt.

> 📸 **Bild-Platzhalter:** `quest-status-in-dialogue-branch-details.png` — Details-Panel des Branch-Nodes mit drei BranchPoints.
> *Setup:* Branch-Node ausgewählt. Details zeigt: `BranchPoints[0]: Description = "Completed", Requirement: HasTag Quest.FindArtifact.Completed`. `BranchPoints[1]: Description = "Active", Requirement: HasTag Quest.FindArtifact.Active`. `BranchPoints[2]: Description = "Nicht gestartet", keine Requirements`.

### 3. Abschluss-Pfad (BranchPoint 0)

SayLine *„Exzellent! Du hast das Artefakt!"* → **ApplyEffect**-Node (`GE_QuestReward`) → Exit (`Completed`).

Damit die Quest nicht doppelt belohnt wird: Am Exit-Node SideEffect `RemoveTag(Quest.FindArtifact.Active)` und `RemoveTag(Quest.FindArtifact.Completed)` – oder das Quest-System reagiert auf `OnDialogueEnded` mit Status `Completed` und räumt auf.

### 4. Aktiv-Pfad (BranchPoint 1)

SayLine *„Du bist noch dran. Das Artefakt liegt südlich vom Turm."* → Exit.

### 5. Nicht-gestartet-Pfad (BranchPoint 2)

SayLine *„Ich brauche deine Hilfe."* → PlayerChoice:
- *„Ja, ich übernehme das."* → SideEffect: `AddTag(Quest.FindArtifact.Active)` am Player → Exit.
- *„Jetzt nicht."* → Exit.

### 6. Compile und testen

Im PIE: ohne Tags → Fallback-Pfad. `Quest.FindArtifact.Active` setzen → mittlerer Pfad. `Quest.FindArtifact.Completed` setzen → oberer Pfad.

## Blueprint-Triggering

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_QuestGiver_Artifact
   └─ ...
```

> 📸 **Bild-Platzhalter:** `quest-status-in-dialogue-bp-trigger.png` — Blueprint mit Start Dialogue und Tags im ASC-Debugger.
> *Setup:* Questgeber-BP mit Start-Dialogue. Daneben: GAS-Debugger Panel mit `Quest.FindArtifact.Active` am Player-ASC markiert.

## Variation / Weiter gehen

- Quest-Status nicht als Tag, sondern als Participant-Variable tracken → [Beziehungs-Score tracken](relationship-counter.md) zeigt das Muster.
- Dialog setzt Quest-Tags selbst → [Dialog setzt Quest-Progress](dialogue-sets-quest-progress.md).
- Belohnung im Dialog ausgeben → [GameplayEffect aus Dialog anwenden](apply-gameplay-effect.md).

## Troubleshooting

**Abschluss-Pfad nie erreicht, obwohl Completed-Tag gesetzt.**
BranchPoints in falscher Reihenfolge: `Active` vor `Completed`. Swap die Reihenfolge.

**Fallback-Pfad läuft obwohl Quest aktiv.**
`bCheckOnInstigator = false` → Tag wird am NPC geprüft. Oder Tag-Name-Tippfehler.

**Belohnung kommt mehrfach.**
Completed-Tag bleibt am Spieler nach dem Dialog. Am Exit-Node RemoveTag hinzufügen oder Quest-System nach `OnDialogueEnded` aufräumen lassen.
