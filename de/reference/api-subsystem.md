---
description: Alle public Methoden und Delegates von UMayDialogueSubsystem.
---

# API: UMayDialogueSubsystem

Zentraler Orchestrator. Ein `UWorldSubsystem`, pro Welt genau eine Instanz. Implementiert außerdem `FTickableGameObject` (für Auto-Advance/Watchdog) und `IMayDialogueBridge` (für externe Konsumenten).

- **Blueprint-Zugriff**: Im Blueprint-Editor unter Kategorie *MayDialogue|Subsystem*.
- **Modul**: `MayDialogue`
- **Base**: `UWorldSubsystem`

---

## Zugriff

```cpp
// C++ — drei äquivalente Wege
UMayDialogueSubsystem* Sub = GetWorld()->GetSubsystem<UMayDialogueSubsystem>();

// Statischer Convenience-Accessor (1.0)
UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(this);

// Via Library-Helper
UMayDialogueSubsystem* Sub = UMayDialogueLibrary::GetDialogueSubsystem(this);
```

```text
// Blueprint
[Get MayDialogue Subsystem]    ← Library-Node, oder
[UMayDialogueSubsystem::Get]   ← direkter Static-Node (DisplayName: "Get MayDialogue Subsystem")
```

### UMayDialogueSubsystem::Get (1.0)

```cpp
UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem",
    meta = (WorldContext = "WorldContext", DisplayName = "Get MayDialogue Subsystem"))
static UMayDialogueSubsystem* Get(const UObject* WorldContext);
```

Null-sicherer statischer Accessor. Gibt `nullptr` zurück, wenn `WorldContext` null ist, keine Welt hat oder das Subsystem nicht vorhanden ist. Äquivalent zu `UMayDialogueLibrary::GetDialogueSubsystem(WorldContext)`.

```cpp
// Null-sicheres Muster
if (auto* S = UMayDialogueSubsystem::Get(this))
{
    S->StartDialogue(Asset, Instigator, Target);
}
```

> 📸 **Bild-Platzhalter:** `subsystem-access-bp.png` — Blueprint-Node `Get MayDialogue Subsystem` mit Rückgabe-Pin.
> *Setup:* Beliebiges Blueprint, Rechtsklick → "Get MayDialogue Subsystem" eingeben. Node mit gelbem Subsystem-Rückgabe-Pin sichtbar. Rückgabe-Pin geht in eine Variable.

---

## Dialog-Lifecycle-Methoden

| Signatur | Rückgabe | Beschreibung |
|---|---|---|
| `StartDialogue(Asset, Instigator, Target)` | `UMayDialogueInstance*` | Pre-Flight-Check → Instance erzeugen → Entry starten. `nullptr` bei Fehler. Nur Server. |
| `K2_CanStartDialogue(Asset, Instigator, Target)` | `bool` | Blueprint-Pre-Flight-Check. Gibt `true` zurück wenn `StartDialogue` klappen würde. |
| `K2_AbortDialogue(Instance)` | `void` | Abortet eine bestimmte Instance. No-op bei `nullptr`. Nur Server/Authority. |
| `AbortAllDialogues()` | `void` | Abortet alle aktiven Instances. Nur Server/Authority. |
| `StopDialogue(Instance)` | `void` | **Deprecated** — stattdessen `K2_AbortDialogue` verwenden. |
| `StopAllDialogues()` | `void` | **Deprecated** — stattdessen `AbortAllDialogues` verwenden. |

```cpp
UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem")
UMayDialogueInstance* StartDialogue(UMayDialogueAsset* Asset, AActor* Instigator, AActor* Target);

// Blueprint: "Can Start Dialogue" — Pre-Flight-Validierung (1.0)
UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem",
    DisplayName = "Can Start Dialogue")
bool K2_CanStartDialogue(UMayDialogueAsset* Asset, AActor* Instigator, AActor* Target);

// Blueprint: "Abort Dialogue" (1.0, kanonisches Verb)
UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem",
    meta = (DisplayName = "Abort Dialogue"))
void K2_AbortDialogue(UMayDialogueInstance* Instance);

UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem")
void AbortAllDialogues();

// Deprecated-Aliases — für Quellcode-Kompatibilität behalten, werden in einer zukünftigen Version entfernt
UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem",
    meta = (DeprecatedFunction,
        DeprecationMessage = "Use K2_AbortDialogue instead."))
void StopDialogue(UMayDialogueInstance* Instance);

UFUNCTION(BlueprintCallable, Category = "MayDialogue|Subsystem",
    meta = (DeprecatedFunction,
        DeprecationMessage = "Use AbortAllDialogues instead."))
void StopAllDialogues();
```

### Verb-Konvention (1.0)

MayDialogue verwendet an jeder öffentlichen Schicht ein einziges kanonisches Verb für „Dialog frühzeitig beenden":

| Schicht | Kanonisches Verb |
|---|---|
| `UMayDialogueSubsystem` | `K2_AbortDialogue` / `AbortAllDialogues` |
| `UMayDialogueLibrary` | `AbortDialogue` / `AbortAllDialogues` |
| `UMayDialogueInstance` | `AbortDialogue` (server-autoritativ) |
| `IMayDialogueBridge` | `AbortDialogue` |
| `UMayDialogueParticipant` (Client-Input) | `RequestAbortDialogue` → `ServerAbortConversation` |

- **`Abort*`** — im Server/Authority-Code aufrufen, um einen Dialog zu beenden.
- **`Request*`** — netz-sichere Wrapper auf `UMayDialogueParticipant`; leiten über Server-RPCs weiter, funktionieren identisch auf Clients und Listen-Server-Hosts. Designer verbinden UI-Buttons direkt mit `Request*`.
- **`Server*` / `Client*`** — rohe UE-RPC-Schicht; nicht aus Gameplay-Code aufrufen, stattdessen `Request*` nutzen.
- **`Stop*`** — deprecated Aliases; kompilieren mit Deprecation-Warnung und leiten an `Abort*` weiter.

**StartDialogue-Ablauf:**

```text
StartDialogue gerufen
    │
    ▼
CanStartDialogue?  →  Nein → nullptr
    │ Ja
    ▼
Dialog aktiv?  →  Ja → AbortAllDialogues
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
| `OnAnyDialogueAborted` | `FOnMayDialogueAborted` | Irgendein Dialog wird abgebrochen — feuert vor `OnAnyDialogueEnded`. |

```cpp
UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnAnyDialogueEvent OnAnyDialogueStarted;

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnAnyDialogueEvent OnAnyDialogueEnded;

UPROPERTY(BlueprintAssignable, Category = "MayDialogue|Events")
FOnMayDialogueAborted OnAnyDialogueAborted;
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

## K2_CanStartDialogue — Pre-Flight-Check (1.0)

Verwenden, bevor Interaktions-Prompts angezeigt oder Ressourcen für einen Dialogstart ausgegeben werden:

```text
[Get MayDialogue Subsystem] → [Can Start Dialogue]
  ├─ Asset:      DA_Greeting
  ├─ Instigator: Player Pawn
  └─ Target:     Guard Actor
       │ (bool)
       ▼
[Branch]
  ├─ True  → Interaktions-Prompt anzeigen
  └─ False → Prompt ausblenden (kein Asset / kein Entry / kein Participant)
```

Prüft: Asset gültig und kompiliert, Entry-Point vorhanden, mindestens einer von Instigator/Target trägt eine `UMayDialogueParticipant`. Startet den Dialog **nicht**. Sicher auf Server oder Client aufrufbar.

---

## Sicherheits-Contracts

- Alle Methoden sind **Game-Thread-only**.
- `StartDialogue` mit `nullptr`-Asset loggt Warning und liefert `nullptr` — kein Crash.
- Mehrfach-Abort derselben Instance ist idempotent.
- Während `AbortAllDialogues` darf aus den Delegate-Handlern heraus **nicht** `StartDialogue` gerufen werden (Re-Entrant-Guard; ein Warning wird geloggt und der Call verworfen).
- `StartDialogue` muss server-seitig aufgerufen werden — Clients leiten über `UMayDialogueParticipant::RequestStartDialogue` weiter.

---

## Deinitialisierung (Level-Travel)

UE ruft `Deinitialize()` beim Welt-Wechsel:

1. `AbortAllDialogues()` — alle Instances sauber abbrechen.
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
