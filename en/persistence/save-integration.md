---
description: Wie du MayDialogue in dein eigenes SaveGame-System einbindest.
---

# SaveGame-Integration

Wenn dein Projekt bereits ein SaveGame-System hat, fügst du MayDialogue-Daten nahtlos ein. Die Participant-Komponente ist schon vorbereitet.

## Blueprint-Schnelleinstieg

Für Projekte ohne eigenes SaveGame-System steht `UMayDialogueSaveHelper` bereit — eine Blueprint Function Library mit einfachen Speicher- und Lade-Nodes.

**Speichern** (z.B. beim Verlassen eines Levels):

```text
[Event Level Exit]
    │
    ▼
[Quick Save To Slot]  (Kategorie: MayDialogue|Persistence)
    ├─ WorldContextObject: Self
    ├─ Slot Name:          "AutoSave"
    └─ User Index:         0
         │ Return Value (bool) — true wenn erfolgreich
```

**Laden** (z.B. beim Start des nächsten Levels):

```text
[Event Begin Play]
    │
    ▼
[Does Slot Exist]  (Kategorie: MayDialogue|Persistence)
    ├─ Slot Name: "AutoSave"
    │ True
    ▼
[Quick Load From Slot]  (Kategorie: MayDialogue|Persistence)
    ├─ WorldContextObject: Self
    ├─ Slot Name:          "AutoSave"
    └─ User Index:         0
```

`Quick Save To Slot` sammelt automatisch die `PersistentMemory` aller `UMayDialogueParticipant`-Komponenten in der Welt und schreibt sie in einen UE-SaveGame-Slot. `Quick Load From Slot` stellt sie beim Laden wieder her — Participants werden anhand ihres `ParticipantTag` identifiziert.

Zusätzlich kannst du projektweite Flags (ohne Participant-Bezug) über `Set Global Bool` / `Get Global Bool` und die entsprechenden Int-, Float- und String-Varianten in der Kategorie `MayDialogue|Persistence|Global` speichern.

---

## Erweiterte Integration (C++)

Wenn dein Projekt ein eigenes SaveGame-System hat, fügst du MayDialogue-Daten direkt in deine Save-Pipeline ein. Die Participant-Komponente ist schon vorbereitet.

## Das UPROPERTY(SaveGame)-Flag

Auf `UMayDialogueParticipant`:

```cpp
UPROPERTY(SaveGame)
FInstancedPropertyBag PersistentMemory;
```

Das `SaveGame`-Specifier sagt UE's Serialisierer: "Diese Property beim Serialisieren eines SaveGame-Archives mitschreiben." Du musst nichts weiter tun, solange dein Archiv `ArIsSaveGame = true` setzt.

## Schritt 1 — Beim Speichern

Wenn dein Projekt speichert, iterierst du alle Participants und serialisierst den Actor:

```cpp
// In deinem SaveSystem oder SaveGame-Handler:
TArray<AActor*> Participants;
UGameplayStatics::GetAllActorsWithComponent(World, UMayDialogueParticipant::StaticClass(), Participants);

for (AActor* Actor : Participants)
{
    TArray<uint8> Bytes;
    FMemoryWriter Writer(Bytes);
    FObjectAndNameAsStringProxyArchive Ar(Writer, /*bLoadIfFindFails=*/false);
    Ar.ArIsSaveGame = true;

    Actor->Serialize(Ar);  // PersistentMemory wird mitgeschrieben, weil SaveGame-Flag
    // Bytes in dein SaveData-Struct packen, Schlüssel = Actor-Name oder GUID
}
```

> 📸 **Bild-Platzhalter:** `saveint-blueprint-save.png` — Blueprint-Graph: Alle Actors mit UMayDialogueParticipant iterieren und serialisieren.
> *Setup:* BP-Graph im SaveSystem-Blueprint. `Get All Actors With Component` (ComponentClass = MayDialogueParticipant) → ForEach-Loop → `Save Actor To Bytes`-Funktion (aus eigener Utility-Library). Schleifenausgang führt in ein `SaveData`-Array. Kein MayDialogue-spezifischer Knoten nötig — normaler UE-SaveGame-Pfad.

## Schritt 2 — Beim Laden

Inverse Richtung: die gespeicherten Bytes auf die Actors anwenden.

```cpp
for (auto& [ActorName, Bytes] : SavedData)
{
    AActor* Actor = FindActorByName(World, ActorName);  // deine Lookup-Logik
    if (!Actor) continue;

    FMemoryReader Reader(Bytes);
    FObjectAndNameAsStringProxyArchive Ar(Reader, false);
    Ar.ArIsSaveGame = true;

    Actor->Serialize(Ar);  // PersistentMemory wird wiederhergestellt
}
```

`FInstancedPropertyBag` kennt sein eigenes Schema und deserialisiert sich korrekt, solange das Schema zwischen Save und Load unverändert ist.

## Manuelles Vorgehen (Alternative)

Du musst den Standard-Archive-Pfad nicht nutzen. Du kannst die PropertyBag auch direkt aus dem Participant holen und in dein SaveGame-Struct stecken:

```cpp
// Beim Speichern:
UMayDialogueParticipant* Part = Actor->FindComponentByClass<UMayDialogueParticipant>();
FInstancedPropertyBag Memory = Part->PersistentMemory;
MySaveData.DialogueMemories.Add(Part->ParticipantTag.GetTagName(), Memory);

// Beim Laden:
FInstancedPropertyBag LoadedMemory = MySaveData.DialogueMemories.FindRef(Part->ParticipantTag.GetTagName());
Part->PersistentMemory = LoadedMemory;
```

Das gibt dir volle Kontrolle über den Sync-Zeitpunkt.

> 📸 **Bild-Platzhalter:** `saveint-manual-bp.png` — Blueprint-Graph: Manuelles Get/Set PersistentMemory.
> *Setup:* BP-Graph zeigt `Get Component by Class` (MayDialogueParticipant) → `Get Persistent Memory` → `Add to SaveData Map`. Zweiter Teil: `Load from Map` → `Set Persistent Memory`. Beide Teile im selben Graph, mit Kommentar-Boxen "Beim Speichern" und "Beim Laden" beschriftet.

## Was genau wird gespeichert

Pro Participant wird die `PersistentMemory`-PropertyBag gespeichert — ein typisiertes Dictionary aus Bool-, Int-, Float-, String- und GameplayTag-Werten. Jede Variable, die du per `SetPersistentBool()`, `SetPersistentInt()` etc. geschrieben hast, lebt dort.

**Nicht** gespeichert werden:

* Laufende Dialog-Instances (sie werden neu gestartet, falls nötig).
* Dialogue-Scope-Variablen.
* UI-State.

## Actor-Identität beim Load

PersistentMemory überlebt nur, wenn die Participant-Komponente beim Load am richtigen Actor landet. Typisches Muster:

1. Level enthält NPC "Guard_01" (in der Szene platziert).
2. Beim Save: Actor-Name oder GUID + PersistentMemory werden gespeichert.
3. Beim Load: Level wird geladen, NPC erscheint wieder, PersistentMemory wird von deinem System über den Namen/GUID zurückgeschrieben.

{% hint style="warning" %}
**Dynamisch gespawnte NPCs:** Wenn ein Actor nicht im Level liegt, sondern dynamisch gespawnt wird, stelle sicher, dass der Spawn-Pfad denselben `ParticipantTag` und eine stabile Identität (z.B. eine gespeicherte GUID) wiederverwendet.
{% endhint %}

## GlobalMemory

`UMayDialogueSaveGame` hat zusätzlich ein `GlobalMemory`-Feld (`FInstancedPropertyBag`). Nutze es für Projekt-weite Dialog-Flags, die keinem bestimmten Participant gehören, z.B. *"Hat der Spieler das Tutorial-Gespräch gesehen?"*. Du greifst per Code auf `UMayDialogueSaveGame::GlobalMemory` zu und speicherst es zusammen mit den Participant-Daten.

> 📸 **Bild-Platzhalter:** `saveint-flow-diagram.png` — Zeitstrahl: User drückt Speichern → System iteriert Participants → Archive serialisiert → Slot geschrieben. Beim Laden: Slot gelesen → Actor gefunden → Archive deserialisiert → PersistentMemory wiederhergestellt.
> *Setup:* Einfaches Flussdiagramm mit zwei Pfaden: "Speichern" (oben) und "Laden" (unten). Speichern: Rechteck "User drückt Speichern" → "Iterate Participants" → "Serialize ArIsSaveGame=true" → "Write Slot". Laden: "Load Slot" → "Find Actor by Name" → "Deserialize" → "PersistentMemory restored". Pfeile deutlich, Schritte nummeriert.
