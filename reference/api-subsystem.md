---
description: Alle public Methoden und Delegates von UMayDialogueSubsystem.
---

# API: UMayDialogueSubsystem

Zentraler Orchestrator. Ein `UWorldSubsystem`, pro Welt genau eine Instanz. Implementiert außerdem `FTickableGameObject` (für Auto-Advance/Watchdog) und `IMayDialogueBridge` (für externe Konsumenten).

- **Header**: `Source/MayDialogue/Public/MayDialogueSubsystem.h`
- **Modul**: `MayDialogue`
- **Base**: `UWorldSubsystem`

---

## Zugriff

```cpp
// C++
UMayDialogueSubsystem* Sub = GetWorld()->GetSubsystem<UMayDialogueSubsystem>();

// Via Library-Helper
UMayDialogueSubsystem* Sub = UMayDialogueLibrary::GetDialogueSubsystem(this);
```

```text
// Blueprint
[Get MayDialogue Subsystem]
```

> 📸 **Bild-Platzhalter:** `subsystem-access-bp.png` — Blueprint-Node `Get MayDialogue Subsystem` mit Rückgabe-Pin.
> *Setup:* Beliebiges Blueprint, Rechtsklick → "Get MayDialogue Subsystem" eingeben. Node mit gelbem Subsystem-Rückgabe-Pin sichtbar. Rückgabe-Pin geht in eine Variable.

---

## Dialog-Lifecycle-Methoden

| Signatur | Rückgabe | Beschreibung |
|---|---|---|
| `StartDialogue(Asset, Instigator, Target)` | `UMayDialogueInstance*` | Pre-Flight-Check → Instance erzeugen → Entry starten. `nullptr` bei Fehler. |
| `CanStartDialogue(Asset, Instigator, Target)` | `bool` | Reiner Query. Klärt ob `StartDialogue` klappen würde. |
| `StopDialogue(Instance)` | `void` | Abortet eine bestimmte Instance. No-op bei `nullptr`. |
| `StopAllDialogues()` | `void` | Abortet alle aktiven Instances. |

```cpp
UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem")
UMayDialogueInstance* StartDialogue(UMayDialogueAsset* Asset, AActor* Instigator, AActor* Target);

UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem")
bool CanStartDialogue(UMayDialogueAsset* Asset, AActor* Instigator, AActor* Target) const;

UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem")
void StopDialogue(UMayDialogueInstance* Instance);

UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem")
void StopAllDialogues();
```

**StartDialogue-Ablauf:**

```text
StartDialogue gerufen
    │
    ▼
CanStartDialogue?  →  Nein → nullptr
    │ Ja
    ▼
Dialog aktiv?  →  Ja → StopAllDialogues
    │
    ▼
Neue Instance erzeugen
    │
    ▼
Participants auflösen
    │
    ▼
OnAnyDialogueStarted broadcasten
    │
    ▼
Entry-Node starten → Instance zurückgeben
```

---

## Query-Methoden

| Signatur | Rückgabe | Beschreibung |
|---|---|---|
| `GetActiveDialogue()` | `UMayDialogueInstance*` | Aktive Instance oder `nullptr`. |
| `IsAnyDialogueActive()` | `bool` | Prüft ob irgendetwas läuft. |

```cpp
UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem")
UMayDialogueInstance* GetActiveDialogue() const;

UFUNCTION(BlueprintPure, Category = "MayDialogue|Subsystem")
bool IsAnyDialogueActive() const;
```

---

## Subsystem-Delegates

| Delegate | Typ | Feuer-Zeitpunkt |
|---|---|---|
| `OnAnyDialogueStarted` | `FOnAnyDialogueEvent` | Direkt nachdem eine Instance startet. |
| `OnAnyDialogueEnded` | `FOnAnyDialogueEvent` | Direkt nachdem eine Instance endet. |

```cpp
UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnAnyDialogueEvent OnAnyDialogueStarted;

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnAnyDialogueEvent OnAnyDialogueEnded;
```

`FOnAnyDialogueEvent` = `DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnAnyDialogueEvent, UMayDialogueInstance*, Instance)`

---

## Read/Write-Methoden (Bridge-API)

Alle direkt am Subsystem als Blueprint-Callable — operieren auf der aktiven Instance.

| Blueprint-Name | Rückgabe | Beschreibung |
|---|---|---|
| `Get Active Dialogue Asset` | `UMayDialogueAsset*` | Asset der aktiven Instance. |
| `Get Current Node GUID` | `FGuid` | GUID des laufenden Nodes. |
| `Get Active Participants` | `TArray<AActor*>` | Alle teilnehmenden Actors. |
| `Get Dialogue Variable (As String)` | `bool` | Liest Dialog-Variable als String. Out-Param: Wert. |
| `Get Participant Variable (As String)` | `bool` | Liest Participant-Variable als String. |
| `Get Pending Choices` | `TArray<FMayDialogueChoiceEntry>` | Aktuelle Choice-Liste (leer wenn kein PlayerChoice). |
| `Set Dialogue Variable (From String)` | `bool` | Setzt Dialog-Variable aus String. |
| `Set Participant Variable (From String)` | `bool` | Setzt Participant-Variable aus String. |
| `Select Choice` | `bool` | Wählt Choice per Index. |
| `Force Advance` | `bool` | Überspringt aktuellen Advance-Wait. |

Details mit Beispielen: [Read/Write-API](../runtime/read-write-api.md).

---

## Sicherheits-Contracts

- Alle Methoden sind **Game-Thread-only**.
- `StartDialogue` mit `nullptr`-Asset loggt Warning und liefert `nullptr` — kein Crash.
- Mehrfach-Abort derselben Instance ist idempotent.
- Während `StopAllDialogues` darf aus den Delegate-Handlern heraus **nicht** `StartDialogue` gerufen werden (Re-Entrant-Guard; ein Warning wird geloggt und der Call verworfen).

---

## Deinitialisierung (Level-Travel)

UE ruft `Deinitialize()` beim Welt-Wechsel:

1. `StopAllDialogues()` — alle Instances sauber abbrechen.
2. Timer und Delegates detachen.
3. `CleanupCompletedDialogues()` final leeren.

Das neue Level bekommt ein frisches Subsystem-Objekt.

---

## Replikation

Das Subsystem ist **nicht repliziert**. Jeder Client hat sein eigenes lokales Subsystem-Objekt. Multiplayer-Sync läuft über `UMayDialogueParticipant`-RPCs.

## Siehe auch

- [API: Library](api-library.md) — Convenience-Wrapper.
- [API: Delegates](api-delegates.md) — vollständige Delegate-Signaturen.
- [Runtime → Subsystem-API](../runtime/subsystem-api.md) — geführter Walkthrough mit Beispielen.
