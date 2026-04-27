---
description: Participant-Komponente am Actor vs. Speaker-Definition im Asset — und wie sie zusammenfinden.
---

# Participants & Sprecher

In MayDialogue gibt es zwei Seiten eines sprechenden Charakters: die **Participant-Komponente** am Actor im Level und den **Speaker-Eintrag** im Dialog-Asset. Beide sind über einen Tag verbunden.

## Das Konzept auf einen Blick

```text
Level (Laufzeit)                 Dialog-Asset (Authoring)
────────────────                 ─────────────────────────
GuardActor                       Speakers-Liste
  └─ UMayDialogueParticipant       └─ "Wächter"
       ParticipantTag:                  SpeakerTag: Dialogue.Speaker.Guard
       Dialogue.Speaker.Guard     ←──── DisplayName: "Wächter"
       DisplayName: "Hans"              Portrait: TX_Guard_Neutral
       PersistentMemory: {...}          NodeColor: #CC3333
```

Die Komponente fragt: *„Wer ist das im Level?"*
Der Speaker fragt: *„Wie wird er in **diesem** Dialog präsentiert?"*

Warum diese Trennung? Ein NPC kann in verschiedenen Dialogen anders heißen und anders aussehen. Derselbe Guard-Actor erscheint im ersten Dialog als anonym „Fremder Wächter" (grau, kein Portrait) und nach der Enthüllung als „Hans, Hauptmann" mit eigenem Portrait — weil jedes Asset seine Sprecher-Einträge frei definiert.

> 📸 **Bild-Platzhalter:** `participant-vs-speaker-diagram.png` — Schaubild: Actor-Seite links, Asset-Seite rechts, Pfeil über SpeakerTag.
> *Setup:* Zwei Kästen nebeneinander. Links: „GuardActor im Level" mit Icons für ActorComponent und einem Tag-Symbol mit Text `Dialogue.Speaker.Guard`. Rechts: „DA_Guard_Greeting (Asset)" mit Speakers-Panel-Ausschnitt: Eintrag „Wächter", SpeakerTag-Feld zeigt `Dialogue.Speaker.Guard`. Ein dicker Pfeil verbindet beide über den Tag. Darunter: kleiner Text „Tag = Brücke zwischen Level und Asset".

## Die Participant-Komponente

`UMayDialogueParticipant` hängst du über den Details-Panel an jeden Dialog-relevanten Actor — NPCs, Spieler-Pawn, Objekte mit Stimme.

### Wichtige Properties

| Property | Zweck |
| --- | --- |
| `ParticipantTag` | Der Match-Key. Convention: `Dialogue.Speaker.<Name>`. |
| `DisplayName` | Fallback-Name, wenn das Asset keinen eigenen definiert. |
| `Portrait` | Fallback-Portrait. Wird vom Asset-Speaker überschrieben. |
| `DefaultDialogue` | Das Asset, das bei Standard-Interaktion startet. |
| `bAutoFacePartner` | Actor dreht sich automatisch zum Gesprächspartner. |
| `CameraTargetOffset` | Offset vom Actor-Origin für CameraFocus-Nodes. |
| `PersistentMemory` | Variablen, die Gespräche überdauern (SaveGame-markiert). |

> 📸 **Bild-Platzhalter:** `participant-component-add.png` — Participant-Komponente zum Actor hinzufügen.
> *Setup:* Details-Panel eines Guard-Actors. „Add Component"-Button angeklickt, Suchfeld zeigt „MayDialogue", der Eintrag `MayDialogueParticipant` ist markiert. Kein sonstiger Content.

> 📸 **Bild-Platzhalter:** `participant-component-filled.png` — Fertig ausgefüllte Participant-Komponente.
> *Setup:* Details-Panel, Abschnitt `MayDialogueParticipant`. Sichtbare Felder: `ParticipantTag = Dialogue.Speaker.Guard`, `DisplayName = Wächter`, `DefaultDialogue = DA_Guard_Greeting` (Asset-Referenz), `bAutoFacePartner = true`, `CameraTargetOffset = (0, 0, 80)`. Portrait-Slot mit Texture2D `TX_Guard_Neutral` befüllt.

### Drei Wege, einen Dialog zu starten

Alle drei Wege führen beim Subsystem zusammen und erzeugen dieselbe Instanz.

**Blueprint — am einfachsten:**

> 📸 **Bild-Platzhalter:** `bp-start-default-dialogue.png` — Blueprint-Graph: StartDefaultDialogue auf der Komponente.
> *Setup:* BP-Graph eines Interaction-Triggers (z.B. Box-Overlap). `Event BeginOverlap` → `Get MayDialogueParticipant` (auf GuardActor) → `Start Default Dialogue`. Pin `Instigator`: `Get Player Pawn`. Alle Nodes sind verbunden und beschriftet.

```cpp
// C++ Variante 1 — auf der Komponente
Guard->FindComponentByClass<UMayDialogueParticipant>()->StartDefaultDialogue(Player);

// C++ Variante 2 — über die Library (Blueprint-nah)
UMayDialogueLibrary::StartDialogue(this, DA_Guard_Greeting, PlayerPawn, GuardActor);

// C++ Variante 3 — direkt am Subsystem
UMayDialogueSubsystem::Get(this)->StartDialogue(DA_Guard_Greeting, PlayerPawn, GuardActor);
```

### Dynamisches Dialog-Swapping

Mit `SetActiveDialogue(NewAsset)` kannst du den Default-Dialog eines NPCs zur Laufzeit wechseln — z.B. wenn eine Quest einen neuen Gesprächsstand freigeschaltet hat.

**Blueprint:**

> 📸 **Bild-Platzhalter:** `bp-set-active-dialogue.png` — Blueprint: SetActiveDialogue auf der Komponente nach einem Quest-Event.
> *Setup:* BP-Graph eines QuestManagers. Event `OnQuestCompleted` → `Get MayDialogueParticipant` (auf GuardActor) → `Set Active Dialogue`. Pin `New Dialogue = DA_Guard_AfterRevelation`. Einfacher linearer Graph.

```cpp
Guard->FindComponentByClass<UMayDialogueParticipant>()->SetActiveDialogue(DA_Guard_AfterRevelation);
```

## Der Sprecher-Eintrag im Asset

Für jeden Sprecher in einem Dialog-Asset trägst du im **Speakers-Panel** einen Eintrag ein:

| Property | Zweck |
| --- | --- |
| `SpeakerTag` | Match-Key — muss mit dem `ParticipantTag` des Actors übereinstimmen. |
| `DisplayName` | Name im UI-Widget. Überschreibt den Fallback der Komponente. |
| `Portrait` | Portrait im UI-Widget. Überschreibt den Fallback der Komponente. |
| `NodeColor` | Title-Bar-Farbe aller SayLine-Nodes dieses Sprechers. |
| `AudioModeOverride` | Default / Spatial3D / Force2D. |
| `VolumeMultiplier` / `PitchMultiplier` | Audio-Tuning pro Sprecher. |

> 📸 **Bild-Platzhalter:** `speakers-panel-filled.png` — Speakers-Panel mit zwei Einträgen.
> *Setup:* Asset-Editor, Speakers-Panel-Tab geöffnet. Zwei Einträge: 1. „Wächter" — SpeakerTag: `Dialogue.Speaker.Guard`, NodeColor: dunkelrot, Portrait: TX_Guard_Neutral. 2. „Spieler" — SpeakerTag: `Dialogue.Speaker.Player`, NodeColor: dunkelblau, kein Portrait. Beide Einträge ausgeklappt.

## Participant-Auflösung zur Laufzeit

Wenn ein SayLine-Node den Speaker-Tag `Dialogue.Speaker.Guard` ansteuert, sucht das Plugin in dieser Reihenfolge nach einem passenden Actor:

1. Instigator-Actor
2. Target-Actor
3. Alle zusätzlich registrierten Participants (z.B. dritter NPC im Gespräch)
4. Fallback: Kein Actor gefunden → Portrait und Name kommen aus dem Asset-Speaker-Eintrag, Audio läuft in 2D.

## PersistentMemory: Was den Dialog überdauert

`PersistentMemory` ist der Ort für Variablen, die nach dem Gespräch weitergelten — z.B. ob der Spieler diesen NPC schon getroffen hat, oder wie viel Vertrauen aufgebaut wurde.

**Blueprint:**

> 📸 **Bild-Platzhalter:** `bp-persistent-memory.png` — Blueprint: SetPersistentBool auf der Participant-Komponente.
> *Setup:* BP-Graph nach einem Dialog-Ende. `OnDialogueEnded` → `Get MayDialogueParticipant` → `Set Persistent Bool`. Pins: `Variable Name = "HasMet"`, `Value = true`. Einfacher Graph, alles beschriftet.

```cpp
// Schreiben
Guard->FindComponentByClass<UMayDialogueParticipant>()->SetPersistentBool("HasMet", true);
Guard->FindComponentByClass<UMayDialogueParticipant>()->SetPersistentInt("FriendshipPoints", 42);

// Lesen mit Default-Wert
bool HasMet     = Part->GetPersistentBool("HasMet", false);
int32 Friendship = Part->GetPersistentInt("FriendshipPoints", 0);
```

Die `PersistentMemory` ist als `SaveGame` markiert. Dein eigenes SaveGame-System packt sie automatisch ein, wenn du die Komponente serialisierst.

## Zusammenfassung

- **Participant-Komponente** = die Identität des Actors im Level. Tag, Gedächtnis, Audio-Overrides, Kamera-Offset.
- **Speaker-Eintrag** = die Präsentation im Dialog-Asset. Name, Portrait, Farbe, Audio-Tuning.
- **Tag-Matching** verbindet beide zur Laufzeit.
- `DefaultDialogue` und `SetActiveDialogue` steuern, welches Gespräch ein NPC startet.
- `PersistentMemory` speichert gesprächsübergreifende Variablen.

Weiter: [Variablen & Scopes](variables-scopes.md).
