---
description: Laufende Dialoge in den Spielstand speichern und exakt an der richtigen Stelle fortsetzen.
---

# Resume-at-Node: Dialoge speichern & fortsetzen

Ein Spieler steht mitten in einem langen Gespräch — und schließt das Spiel. Beim nächsten Start soll der Dialog dort weitergehen, wo er aufgehört hat: dieselbe Zeile, dieselben Variablen, dieselbe offene Auswahl. Genau das macht **Resume-at-Node**.

MayDialogue serialisiert eine laufende Instance in einen kompakten **Snapshot** und kann sie später wieder herstellen. Du musst dafür nichts von Hand verdrahten: Der QuickSave-Helper nimmt aktive Dialoge automatisch mit. Wer ein eigenes SaveGame-System hat, ruft eine einzige Funktion auf.

{% hint style="info" %}
Resume-at-Node speichert den **Fortschritt** eines Gesprächs, nicht nur die Participant-Memory. Die Participant-Memory (`PersistentMemory`) lebt unabhängig davon weiter — siehe [QuickSave-Helper](../persistence/quicksave-helper.md) und [SaveGame-Integration](../persistence/save-integration.md).
{% endhint %}

## Der Null-Code-Weg: QuickSave nimmt Dialoge mit

Wenn du den [QuickSave-Helper](../persistence/quicksave-helper.md) nutzt, ist Resume bereits eingebaut. `QuickSaveToSlot` erfasst alle gerade laufenden Dialoge mit, `QuickLoadFromSlot` setzt sie nach dem Laden automatisch fort.

**Speichern** — beim Verlassen des Levels oder über ein Pause-Menü:

```text
[Event "Spieler speichert"]
    │
    ▼
[Quick Save To Slot]   (Kategorie: MayDialogue|Persistence)
    ├─ World Context: Self
    ├─ Slot Name:     "AutoSave"
    └─ User Index:    0
         → erfasst PersistentMemory + alle aktiven Dialog-Snapshots
```

**Laden** — beim Start des Levels:

```text
[Event Begin Play]
    │
    ▼
[Does Slot Exist? "AutoSave"]
    ├─ True  → [Quick Load From Slot "AutoSave"]
    │             → stellt PersistentMemory wieder her
    │             → setzt jeden gespeicherten Dialog am gemerkten Node fort
    └─ False → (kein Save vorhanden — erster Start)
```

Mehr ist nicht nötig. Der Spieler landet wieder in der zuletzt gezeigten Zeile, und wenn er beim Speichern vor einer Auswahl stand, wird genau diese Auswahl erneut präsentiert.

{% hint style="warning" %}
**Laden ist ein *Wiederherstellen*, kein *Überlagern* (Verhaltensänderung).** Wenn `QuickLoadFromSlot` die `PersistentMemory` eines Participants wiederherstellt, wird die Live-Memory jetzt **zuerst geleert** und anschließend *exakt* der gespeicherte Bag reproduziert — ein Ersetzen, keine Vereinigung. Das ist wichtig, wenn du einen *älteren* Save nach späterem Spielverlauf lädst: Schlüssel, die nach dem Speichern geschrieben wurden (inklusive der automatisch geschriebenen `MayDlg_Visit_*`-Fortschrittszähler oder beliebiger `SetPersistent*`-Aufrufe aus dem späteren Verlauf), werden **verworfen** — das Laden eines älteren Saves rollt also wirklich auf diesen Stand zurück, statt späteren Fortschritt stillschweigend zu behalten. Der Reset ist begrenzt: Nur Participants, die **im Save vorhanden** sind, werden zurückgesetzt — ein Participant ohne gespeicherten Eintrag behält seinen Live-Zustand unverändert (QuickLoad für ein Tag ohne gespeicherte Daten ist ein No-op). Über die `SetGlobal*`-Helfer gesetzte `GlobalMemory` bleibt über ein Save hinweg ebenfalls erhalten.
{% endhint %}

{% hint style="warning" %}
Resume ist eine **Server-Operation**. In Multiplayer-Projekten läuft Speichern und Fortsetzen ausschließlich auf dem Server (so wie der Dialog selbst nur auf dem Server lebt — siehe [Subsystem-API](subsystem-api.md)). Clients bekommen den fortgesetzten Dialog über die normalen Client-RPCs gespiegelt, sobald der Server ihn wieder aufnimmt.
{% endhint %}

> 📸 **Bild-Platzhalter:** `resume-quicksave-bp.png` — Blueprint-Graph: Pause-Menü-Button → QuickSaveToSlot, daneben BeginPlay → DoesSlotExist → QuickLoadFromSlot.
> *Setup:* BP-Graph im GameMode oder Level-Blueprint. Linker Bereich (Kommentar "Speichern"): `On Save Clicked` → `Quick Save To Slot` (Slot="AutoSave"). Rechter Bereich (Kommentar "Laden"): `Event Begin Play` → `Does Slot Exist` → `Branch` → True: `Quick Load From Slot`. Während ein Dialog im Hintergrund läuft (sichtbares Dialog-Widget im Viewport-Hintergrund), damit klar wird, dass der laufende Dialog miterfasst wird.

## Was genau fortgesetzt wird — und was nicht

Resume stellt den **logischen Zustand** des Gesprächs wieder her, nicht jede flüchtige Laufzeit-Ressource. Das ist Absicht: ein neu gestartetes Spiel hat keine alten Timer, keine laufenden Audioquellen und keine offenen Montage-Delegates mehr.

| Wird wiederhergestellt | Wird **nicht** wiederhergestellt |
| --- | --- |
| Aktuelle Node-Position (`CurrentNodeGuid`) | Laufende Timer (z.B. Auto-Advance-Countdown) |
| Dialogue-Scope-Variablen (`DialogueVariables`) | Voice-/Audio-Wiedergabe (startet nicht in der Mitte neu) |
| Scope-Stack (Sub-Dialog-Verschachtelung) | Async-Node-Zustand (Wait, Montage, Timer) |
| Offene Auswahl (wird erneut präsentiert) | UI-Animationen / Typewriter-Position |
| Participant-Tags der Beteiligten | |

**Async-Nodes starten ihren Node frisch.** Stand der Spieler beim Speichern auf einem Node, der gerade auf etwas wartet (ein `Wait`-Node, eine `PlayAnimation` mit *Wait For Montage End*, ein laufender Auto-Advance-Timer), dann wird dieser Node beim Fortsetzen **neu betreten** — nicht mitten in seiner Wartephase eingefroren. Der Timer läuft also von vorn, die Animation startet neu. Für den Spieler fühlt sich das natürlich an: Die Zeile wird sauber von Anfang an gezeigt.

{% hint style="info" %}
**Faustregel:** Resume merkt sich *wo* der Dialog steht und *was er weiß* — nicht *wie weit eine Animation gerade abgespielt war*. Plane Speicherpunkte so, dass das Fortsetzen einer Zeile von vorn akzeptabel ist (in der Praxis fast immer der Fall).
{% endhint %}

## C++ / eigenes SaveGame-System

Hast du ein eigenes SaveGame-System, umgehst du den QuickSave-Helper und arbeitest direkt mit dem Snapshot.

### Snapshot erzeugen

Iteriere die laufenden Instances über `Subsystem->GetAllActiveDialogues()` (mehrere Dialoge können gleichzeitig laufen) und ruf `CreateSnapshot` für jede auf. (Für einen einzelnen Dialog liefert `Subsystem->GetActiveDialogue()` nur den zuletzt gestarteten.)

```cpp
#include "MayDialogueInstance.h"
#include "MayDialogueSaveGame.h"

// Beim Speichern: jede aktive Instance in einen Snapshot schreiben.
UMayDialogueSaveGame* Save = /* dein Save-Objekt */;

for (UMayDialogueInstance* Inst : Subsystem->GetAllActiveDialogues())
{
    FMayDialogueInstanceSnapshot Snapshot;
    if (Inst->CreateSnapshot(Snapshot))      // const — verändert die Instance nicht
    {
        Save->ActiveDialogueSnapshots.Add(Snapshot);
    }
}
```

`CreateSnapshot` ist `const` — sie liest die Instance aus, ohne sie zu verändern. Der laufende Dialog wird nicht unterbrochen. Sie liefert `bool`: `false` (und lässt `OutSnapshot` unverändert), wenn es nichts Sinnvolles zu erfassen gibt — der Dialog ist inaktiv, bereits beendet oder hat keinen auflösbaren aktuellen Node bzw. kein Asset. Solche Instances überspringst du, wie die Schleife oben es tut.

### Snapshot fortsetzen

Beim Laden gibst du jeden gespeicherten Snapshot zurück ans Subsystem:

```cpp
UMayDialogueSubsystem* Subsystem = GetWorld()->GetSubsystem<UMayDialogueSubsystem>();

for (const FMayDialogueInstanceSnapshot& Snapshot : Save->ActiveDialogueSnapshots)
{
    // Server-only: stellt eine Instance her und setzt sie am gemerkten Node fort.
    Subsystem->ResumeDialogueFromSnapshot(Snapshot);
}
```

`ResumeDialogueFromSnapshot` lädt das im Snapshot referenzierte `DialogueAsset` (als Soft-Path gespeichert — siehe Tabelle unten), erstellt eine neue Instance, schreibt Variablen und Scope-Stack zurück und transitioniert auf `CurrentNodeGuid`. Stand der Snapshot vor einer Auswahl, wird die Auswahl neu evaluiert und präsentiert.

## Snapshot-Felder

`FMayDialogueInstanceSnapshot` ist das serialisierbare Abbild einer Instance.

| Feld | Typ | Inhalt |
| --- | --- | --- |
| `DialogueAsset` | `FSoftObjectPath` | Welches Asset lief — als Soft-Path, damit das Asset nicht ungewollt im Speicher gehalten wird. Beim Fortsetzen wird es (synchron) nachgeladen. |
| `CurrentNodeGuid` | `FGuid` | Position im Graphen: der Node, an dem fortgesetzt wird. |
| `CurrentChoiceNodeGuid` | `FGuid` | Der PlayerChoice-Node, der aktiv war (damit die Resume-Präsentation und `ReturnToCurrentChoice` den richtigen Node treffen). |
| `PreviousChoiceNodeGuid` | `FGuid` | Der PlayerChoice-Node, der vor dem aktuellen aktiv war (damit `ReturnToLastChoice`-Menüketten nach dem Fortsetzen weiter funktionieren). |
| `DialogueVariables` | `FInstancedPropertyBag` | Die typisierten Dialogue-Scope-Variablen (Bool/Int/Float/String/Tag). |
| `ScopeStack` | `TArray<FMayDialogueScopeSnapshotEntry>` | Sub-Dialog-Verschachtelung für Link/Return. Jeder Frame speichert das Eltern-Asset als Soft-Path plus dessen Return-Node-GUID. |
| `NodeVisitCounts` | `TMap<FGuid, int32>` | Pro-Session-Besuchszähler der Nodes, damit Visit-Count-Gate-Requirements nach dem Fortsetzen identisch auflösen. |
| `InstigatorParticipantTag` | `FGameplayTag` | ParticipantTag des Instigators, zum Wiederanbinden an einen lebenden Actor beim Fortsetzen. |
| `TargetParticipantTag` | `FGameplayTag` | ParticipantTag des Targets, zum Wiederanbinden an einen lebenden Actor beim Fortsetzen. |
| `bWasWaitingForChoice` | `bool` | True, wenn die Instance zum Erfassungszeitpunkt auf eine Spielerauswahl wartete (steuert, ob Resume die Auswahl erneut präsentiert oder den Node direkt wieder betritt). |

Die **offene Auswahl-Liste selbst wird nicht gespeichert** — beim Fortsetzen wird der gespeicherte PlayerChoice-Node erneut betreten und präsentiert eine frisch evaluierte Auswahl-Liste, sodass veralteter Auswahl-Text nie wieder auftauchen kann.

## API-Referenz

| Symbol | Signatur | Zweck |
| --- | --- | --- |
| `FMayDialogueInstanceSnapshot` | `USTRUCT` | Serialisierbares Abbild einer laufenden Instance (Felder siehe oben). |
| `UMayDialogueInstance::CreateSnapshot` | `bool CreateSnapshot(FMayDialogueInstanceSnapshot& OutSnapshot) const` | Schreibt den aktuellen Zustand in `OutSnapshot`. `const` — verändert die Instance nicht. Liefert `false` (und lässt `OutSnapshot` unverändert), wenn nichts Sinnvolles erfasst werden kann. |
| `UMayDialogueSubsystem::ResumeDialogueFromSnapshot` | `UMayDialogueInstance* ResumeDialogueFromSnapshot(const FMayDialogueInstanceSnapshot&)` | **Nur Server.** Stellt eine Instance her und setzt sie am gemerkten Node fort. |
| `UMayDialogueSaveGame::ActiveDialogueSnapshots` | `TArray<FMayDialogueInstanceSnapshot>` | Die im Slot gespeicherten Dialog-Snapshots. |
| `UMayDialogueSaveGame::SchemaVersion` | `int32` (jetzt **2**) | On-Disk-Schema-Version. Wird auf 2 angehoben, weil das Save jetzt Dialog-Snapshots enthält. |
| `UMayDialogueSaveHelper::QuickSaveToSlot` | `bool QuickSaveToSlot(WorldContext, SlotName, UserIndex)` | Erfasst PersistentMemory **und** aktive Dialog-Snapshots. |
| `UMayDialogueSaveHelper::QuickLoadFromSlot` | `bool QuickLoadFromSlot(WorldContext, SlotName, UserIndex)` | Stellt PersistentMemory wieder her **und** setzt gespeicherte Dialoge fort. |

## Upgrade & Kompatibilität

Das `SchemaVersion`-Feld in `UMayDialogueSaveGame` steigt von `1` auf `2`, weil das Save jetzt Dialog-Snapshots aufnimmt.

{% hint style="success" %}
**Alte Saves laden weiterhin.** Ein vor dem Update geschriebener Slot (Schema 1) enthält keine `ActiveDialogueSnapshots` — die Migration lädt ihn problemlos, das Snapshot-Array bleibt leer, die Participant-Memory wird normal wiederhergestellt. Es geht kein Spielstand verloren; lediglich ein zum Zeitpunkt des alten Saves laufender Dialog kann naturgemäß nicht fortgesetzt werden (er wurde damals nicht erfasst).
{% endhint %}

## Stolperfallen

| Fall | Was passiert / zu beachten |
| --- | --- |
| **Asset umbenannt/verschoben** | `DialogueAsset` ist ein Soft-Path. Verschiebst du das Dialog-Asset zwischen Save und Load, findet das Resume es nicht mehr — halte Asset-Pfade stabil oder nutze Redirektoren. |
| **Node aus dem Graph gelöscht** | Wurde der Node mit `CurrentNodeGuid` zwischen Save und Load entfernt (Graph editiert + neu kompiliert), kann das Resume nicht andocken. Das Subsystem bricht den betroffenen Snapshot sauber ab. |
| **Participant fehlt beim Load** | Werden die im Snapshot referenzierten Participants nicht wieder in die Welt geladen, kann der Dialog seine Beteiligten nicht auflösen — sorge dafür, dass die NPCs vor dem Resume existieren (vgl. Actor-Identität in der [SaveGame-Integration](../persistence/save-integration.md)). |
| **Mitten in einem Async-Node gespeichert** | Erwartet, dass der Node frisch neu startet (siehe oben) — kein Bug, sondern Design. |
| **Resume auf einem Client aufgerufen** | `ResumeDialogueFromSnapshot` ist server-only und no-op auf `NM_Client`. Treibe Save/Load über deinen Server-/Save-System-Pfad. |

> 📸 **Bild-Platzhalter:** `resume-snapshot-flow.png` — Flussdiagramm: Aktive Instance → CreateSnapshot → ActiveDialogueSnapshots im SaveGame → Slot. Beim Laden: Slot → ResumeDialogueFromSnapshot → neue Instance am CurrentNodeGuid.
> *Setup:* Einfaches zweispuriges Flussdiagramm. Oben "Speichern": Rechteck "Laufende Instance" → "CreateSnapshot (const)" → "ActiveDialogueSnapshots" → "Slot geschrieben (Schema 2)". Unten "Laden": "Slot gelesen" → "ResumeDialogueFromSnapshot (Server)" → "Neue Instance" → "Transition auf CurrentNodeGuid" → "Offene Auswahl erneut präsentiert". Pfeile deutlich, Schritte nummeriert.
