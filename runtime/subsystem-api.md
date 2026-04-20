# Subsystem-API

`UMayDialogueSubsystem` ist ein `UWorldSubsystem`, das zusätzlich `FTickableGameObject` und `IMayDialogueBridge` implementiert. Eine Instanz pro UE-Welt.

## Zugriff

```cpp
// C++
UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(World);

// Blueprint
Get Game Instance → Get World → Get Subsystem (MayDialogue)
```

Oder kompakter per Library:

```cpp
UMayDialogueLibrary::GetDialogueSubsystem(WorldContext);
```

## Lifecycle-Methoden

```cpp
UMayDialogueInstance* StartDialogue(UMayDialogueAsset* Asset, AActor* Instigator, AActor* Target);
bool                  CanStartDialogue(UMayDialogueAsset* Asset, AActor* Instigator, AActor* Target) const;
void                  StopDialogue(UMayDialogueInstance* Instance);
void                  StopAllDialogues();
```

| Methode | Wirkung |
| --- | --- |
| `StartDialogue` | Pre-Flight-Check → Instance erzeugen → Start am Entry. Liefert die Instance oder nullptr bei Failure. |
| `CanStartDialogue` | Reiner Query; gibt an, ob ein Start klappen würde. |
| `StopDialogue` | Bricht eine bestimmte Instance ab. |
| `StopAllDialogues` | Bricht alle aktiven Instances ab. |

## Query-Methoden

```cpp
UMayDialogueInstance* GetActiveDialogue() const;   // aktive (neueste) oder nullptr
bool                  IsAnyDialogueActive() const;
```

## Tick

Das Subsystem implementiert `FTickableGameObject::Tick(...)`. Verantwortlich für:

* **Camera-Blend-Fortschritt** (während aktiver CameraFocus-Blends).
* **Auto-Advance-Timer** (Advance-Mode `Timer`).
* **Choice-Timeout**.
* **Async-Node-Watchdogs**.
* **Cleanup** von beendeten Instanzen am Frame-Ende.

Du musst nie selbst ticken – alles läuft automatisch.

## Subsystem-Delegates

```cpp
UPROPERTY(BlueprintAssignable)
FOnAnyDialogueEvent OnAnyDialogueStarted;

UPROPERTY(BlueprintAssignable)
FOnAnyDialogueEvent OnAnyDialogueEnded;
```

Beide feuern **pro Instance** – nicht nur pro aktive. Wenn parallel mehrere Instances starten und enden (was auf einer Welt selten vorkommen sollte, da das Subsystem Single-Active erzwingt), bekommst du für jede einzelne ein Event.

## IMayDialogueBridge-Implementierung

Das Subsystem implementiert das `IMayDialogueBridge`-Interface (siehe [Bridge & Lifecycle-Events](bridge-events.md)):

```cpp
virtual UMayDialogueAsset* GetActiveDialogueAsset() const;
virtual FGuid              GetCurrentNodeGUID() const;
virtual TArray<AActor*>    GetActiveParticipants() const;
virtual bool               GetDialogueVariable(FName, EMayDialogueVariableType, FString& OutValueAsString) const;
// ... etc.
```

Externe Systeme können MayDialogue also ausschließlich über das `IMayDialogueBridge`-Interface konsumieren, ohne direkt auf den Subsystem-Typ zu zeigen.

## Typische Blueprint-Nutzung

```
Get May Dialogue Subsystem
  │
  ├→ Is Any Dialogue Active?  (bool)
  │
  ├→ Get Active Dialogue       (UMayDialogueInstance)
  │
  └→ Bind Event to On Any Dialogue Started / Ended
```

## Replication

Das Subsystem ist aktuell **server-seitig**. Multiplayer-Synchronisation erfolgt über Participant-RPCs (siehe [Participants & Sprecher](../concepts/participants-speakers.md#netz-awareness-multiplayer-ready)). Clients erhalten ihre Messages über den lokalen Participant, nicht direkt vom Subsystem.

## Anmerkungen

* Das Subsystem ist **nicht repliziert** – jeder Client hat sein eigenes Subsystem-Objekt mit lokalem State.
* Beim Level-Travel wird das Subsystem zerstört; `Deinitialize()` ruft automatisch `StopAllDialogues()` auf.
