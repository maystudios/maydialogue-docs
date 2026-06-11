---
description: Quest-Status per GameplayTag im Dialog abfragen und verschiedene Gesprächspfade ausliefern.
---

# Quest-Status im Dialog lesen

## Szenario

Ein Questgeber reagiert unterschiedlich, je nachdem ob der Spieler die zugehörige Quest noch nicht hat, sie aktiv hat oder sie abgeschlossen hat. Die Quest-Status werden als GameplayTags am Spieler-ASC geführt. Der Dialog-Branch liest diese Tags und wählt den passenden Pfad.

## Was du lernst

- Zwei Branch-Nodes verketten, um drei Quest-Zustände zu routen.
- Die Checks korrekt anordnen (Completed vor Active).
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

> 📸 **Bild-Platzhalter:** `quest-status-in-dialogue-graph-overview.png` — Asset-Editor mit zwei verketteten Branch-Nodes.
> *Setup:* Asset `DA_QuestGiver_Artifact` geöffnet. Entry → Branch A (Condition: HasTag Completed). Branch A True → kurze Kette SayLine + ApplyEffect → Exit. Branch A False → Branch B (Condition: HasTag Active). Branch B True → SayLine → Exit. Branch B False → SayLine → PlayerChoice → zwei Exits. Alle drei Pfade gut lesbar auseinandergezogen.

## Schritt-für-Schritt

### 1. Quest-Tags definieren

In `DefaultGameplayTags.ini`:
```ini
+GameplayTagList=(Tag="Quest.FindArtifact.Active")
+GameplayTagList=(Tag="Quest.FindArtifact.Completed")
```

### 2. Zwei verkettete Branch-Nodes (drei Ausgänge)

Ein Branch wertet eine einzelne `Condition` aus und routet nach **True** / **False** (plus optionalem Default). Für ein dreifaches *completed / active / not-started*-Split **verkette zwei Branch-Nodes**:

**Branch A** — `Condition` = HasTag `Quest.FindArtifact.Completed`, `bCheckOnInstigator: true`.
- **True** → Abschluss-Pfad (Schritt 3).
- **False** → **Branch B**.

**Branch B** — `Condition` = HasTag `Quest.FindArtifact.Active`, `bCheckOnInstigator: true`.
- **True** → Aktiv-Pfad (Schritt 4).
- **False** → Nicht-gestartet-Pfad (Schritt 5).

Die Reihenfolge ist entscheidend: `Completed` **zuerst** prüfen (Branch A), da ein Spieler nach Abschluss möglicherweise beide Tags trägt — sonst würde der Aktiv-Pfad eine abgeschlossene Quest abfangen.

> 📸 **Bild-Platzhalter:** `quest-status-in-dialogue-branch-details.png` — Zwei verkettete Branch-Nodes im Graph.
> *Setup:* Entry → Branch A (Condition: HasTag Quest.FindArtifact.Completed). Branch A True → Abschluss-Pfad; Branch A False → Branch B (Condition: HasTag Quest.FindArtifact.Active). Branch B True → Aktiv-Pfad; Branch B False → Nicht-gestartet-Pfad. Alle Verbindungen sichtbar.

### 3. Abschluss-Pfad (Branch A → True)

SayLine *„Exzellent! Du hast das Artefakt!"* → **ApplyEffect**-Node (`GE_QuestReward`) → Exit (`Completed`).

Damit die Quest nicht doppelt belohnt wird: Am Exit-Node SideEffect `RemoveTag(Quest.FindArtifact.Active)` und `RemoveTag(Quest.FindArtifact.Completed)` – oder das Quest-System reagiert auf `OnDialogueEnded` mit Status `Completed` und räumt auf.

### 4. Aktiv-Pfad (Branch B → True)

SayLine *„Du bist noch dran. Das Artefakt liegt südlich vom Turm."* → Exit.

### 5. Nicht-gestartet-Pfad (Branch B → False)

SayLine *„Ich brauche deine Hilfe."* → PlayerChoice:
- *„Ja, ich übernehme das."* → SideEffect: `AddTag(Quest.FindArtifact.Active)` am Player → Exit.
- *„Jetzt nicht."* → Exit.

### 6. Compile und testen

Im PIE: ohne Tags → Nicht-gestartet-Pfad. `Quest.FindArtifact.Active` setzen → Aktiv-Pfad. `Quest.FindArtifact.Completed` setzen → Abschluss-Pfad.

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
Die Branches sind in falscher Reihenfolge verkettet: `Active` (Branch A) vor `Completed`. Mach den **Completed**-Check zum ersten Branch, da eine abgeschlossene Quest beide Tags tragen kann.

**Nicht-gestartet-Pfad läuft obwohl Quest aktiv.**
`bCheckOnInstigator = false` → Tag wird am NPC geprüft. Oder Tag-Name-Tippfehler.

**Belohnung kommt mehrfach.**
Completed-Tag bleibt am Spieler nach dem Dialog. Am Exit-Node RemoveTag hinzufügen oder Quest-System nach `OnDialogueEnded` aufräumen lassen.
