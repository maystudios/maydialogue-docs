---
description: Participant-Memory speichern und laden ohne eigenes SaveGame-System.
---

# QuickSave-Helper

Für Projekte ohne eigenes SaveGame-System liefert MayDialogue `UMayDialogueSaveHelper` — eine Blueprint Function Library mit vier Funktionen. Damit speicherst und lädst du die PersistentMemory aller Participants mit einem einzigen Funktionsaufruf.

## Die Kernfunktionen

| Funktion | Was sie tut |
| --- | --- |
| `QuickSaveToSlot(SlotName, UserIndex)` | Sammelt PersistentMemory aller Participants in der Welt, schreibt in Slot. |
| `QuickLoadFromSlot(SlotName, UserIndex)` | Liest Slot, verteilt Memory an alle Participants mit passendem ParticipantTag. |
| `DeleteSlot(SlotName, UserIndex)` | Löscht einen Slot. |
| `DoesSlotExist(SlotName, UserIndex)` | Prüft ob ein Slot vorhanden ist (vor Load aufrufen). |

Alle sind in Blueprint verfügbar (Kategorie `MayDialogue|Persistence`).

## GlobalMemory-Helfer

Für projekt-weite Flags (z.B. "Hat der Spieler das Intro gesehen?") stehen einfache Getter/Setter auf dem `GlobalMemory`-Feld bereit — ohne direkten Zugriff auf den `FInstancedPropertyBag`-Container:

| Funktion | Typ | Beschreibung |
| --- | --- | --- |
| `GetGlobalBool(SlotName, Key, Default)` | `bool` | Bool-Flag lesen |
| `SetGlobalBool(SlotName, Key, Value)` | — | Bool-Flag schreiben |
| `GetGlobalInt(SlotName, Key, Default)` | `int32` | Integer-Wert lesen |
| `SetGlobalInt(SlotName, Key, Value)` | — | Integer-Wert schreiben |
| `GetGlobalFloat(SlotName, Key, Default)` | `float` | Float-Wert lesen |
| `SetGlobalFloat(SlotName, Key, Value)` | — | Float-Wert schreiben |
| `GetGlobalString(SlotName, Key, Default)` | `FString` | String-Wert lesen |
| `SetGlobalString(SlotName, Key, Value)` | — | String-Wert schreiben |

Alle GlobalMemory-Helfer sind Blueprint-Callable (Kategorie `MayDialogue|Persistence|Global`).

```text
[Set Global Float]   (z.B. beim Companion-Dialog-Ende)
  ├─ Slot Name: "AutoSave"
  ├─ Key:       "CompanionAffection"
  └─ Value:     0.75
```

> **Hinweis:** `ParticipantMemory` und `GlobalMemory` sind auf dem `UMayDialogueSaveGame`-Objekt selbst nicht direkt BP-zugänglich (`FInstancedPropertyBag`-Limitation). Die GlobalMemory-Helfer und `GetSavedParticipantTags()` decken die üblichen BP-Zugriffspfade ab.

> 📸 **Bild-Platzhalter:** `quicksave-blueprint-save.png` — Blueprint-Graph: Level-Exit-Event → QuickSaveToSlot.
> *Setup:* BP-Graph im Level-Blueprint. `Event Level Exit` → `Does Slot Exist` (SlotName="AutoSave") → `Quick Save To Slot` (WorldContextObject=Self, SlotName="AutoSave", UserIndex=0) → `Print String "Gespeichert"`. Alle Nodes verbunden, Rückgabewert (bool) ignoriert via Dummy-Branch.

## Speichern — Beispiel

```text
[Level-Exit-Event]
  │
  └─ [Quick Save To Slot]
       SlotName:  "AutoSave"
       UserIndex: 0
       → gibt bool (Erfolg) zurück
```

## Laden — Beispiel

```text
[Begin Play]
  │
  ├─ [Does Slot Exist? "AutoSave"]
  │     ├─ true  → [Quick Load From Slot "AutoSave"]
  │     └─ false → (keine Aktion — erster Start)
```

> 📸 **Bild-Platzhalter:** `quicksave-blueprint-load.png` — Blueprint-Graph: BeginPlay → DoesSlotExist → QuickLoadFromSlot mit Branch.
> *Setup:* BP-Graph im GameMode oder LevelBlueprint. `Event Begin Play` → `Does Slot Exist` (SlotName="AutoSave") → `Branch`. True-Pfad: `Quick Load From Slot` (SlotName="AutoSave"). False-Pfad: keine Node (leer / "first run"-Kommentar). Klarer Pfad, gut lesbar.

## Wie der Match funktioniert

`QuickLoadFromSlot` verteilt Memory anhand des `ParticipantTag`. Participants im gespeicherten Slot und Participants in der aktuell geladenen Welt werden per Tag-Name abgeglichen. Nur Participants, deren `ParticipantTag` im Slot vorhanden ist, erhalten ihre Daten zurück.

{% hint style="warning" %}
**ParticipantTag muss stabil sein.** Wenn du den `ParticipantTag` eines NPCs zwischen Save und Load änderst, findet der Load-Pfad keinen Match. Halte Tags konsistent.
{% endhint %}

## Einschränkungen

| Einschränkung | Auswirkung |
| --- | --- |
| Nur MayDialogue-Daten | Inventar, Spielerposition etc. werden nicht gespeichert |
| Kein Versioning | Nach einem Update der PropertyBag-Struktur können alte Slots inkompatibel sein |
| Kein Delta-Save | Jeder Save schreibt alle Participant-Daten neu |
| Kein Multi-Slot-Management | Du organisierst mehrere Slots selbst per SlotName |

## Wann QuickSave-Helper, wann eigenes System?

| Szenario | Empfehlung |
| --- | --- |
| Game-Jam oder Prototyp | QuickSave-Helper — sofort einsatzbereit |
| Kleines Indie-Projekt ohne komplexen State | QuickSave-Helper als Start, migrieren wenn nötig |
| Projekt mit Inventar, Weltstate, Quests | Eigenes SaveGame-System, `PersistentMemory` per `ArIsSaveGame` einbetten (→ [SaveGame-Integration](save-integration.md)) |

## Gespeicherte Participant-Tags lesen

Mit `GetSavedParticipantTags(SlotName, UserIndex)` (BlueprintPure) kannst du aus einem Slot die Liste aller gespeicherten Participant-Tags abrufen — nützlich für Slot-Preview-UIs (z.B. "Wieviele NPCs wurden schon gesprochen?"):

```text
[Get Saved Participant Tags]
  ├─ Slot Name: "AutoSave"
  ├─ User Index: 0
  └─ Return: TArray<FGameplayTag>
```

> 📸 **Bild-Platzhalter:** `quicksave-slot-naming.png` — Blueprint-Graph: dynamischer SlotName aus "Save_" + CurrentLevel.
> *Setup:* BP-Graph. `Get Current Level Name` → `Append "Save_"` → `Quick Save To Slot` mit dem zusammengesetzten SlotName. Zeigt, wie du pro Level einen eigenen Slot nutzen kannst.
