---
description: Alle Methoden von UMayDialogueLibrary — Signaturen, Parameter, Rückgabe, Verwendungszweck.
---

# API: UMayDialogueLibrary

Blueprint Function Library mit statischen Methoden. Convenience-Layer über dem Subsystem — kein eigener State, keine eigenen Delegates.

- **Header**: `Source/MayDialogue/Public/MayDialogueLibrary.h`
- **Modul**: `MayDialogue`
- **Blueprint-Kategorie**: `MayDialogue`

> 📸 **Bild-Platzhalter:** `library-node-palette.png` — Blueprint-Node-Palette mit allen Library-Nodes.
> *Setup:* Beliebiges Blueprint öffnen, Rechtsklick im Graphen, Suchbegriff "MayDialogue" eingeben. Screenshot der Palette mit allen sichtbaren Nodes: Start Dialogue, Stop Dialogue, Stop All Dialogues, Get Active Dialogue, Is Any Dialogue Active, Get Dialogue Subsystem. Kategorie-Header "MayDialogue" sichtbar.

---

## Methoden-Überblick

| Funktion | Art | Parameter | Rückgabe | Wann nutzen |
|---|---|---|---|---|
| `Start Dialogue` | Callable | `WorldContext`, `Asset`, `Instigator`, `Target` | `UMayDialogueInstance*` | Dialog starten aus beliebigem Blueprint. |
| `Stop Dialogue` | Callable | `Instance` | — | Eine bestimmte Instance stoppen. |
| `Stop All Dialogues` | Callable | `WorldContext` | — | Alle Dialoge abbrechen (Tod, Level-Wechsel). |
| `Get Active Dialogue` | Callable | `WorldContext` | `UMayDialogueInstance*` | Aktive Instance holen um State zu lesen. |
| `Is Any Dialogue Active` | Pure | `WorldContext` | `bool` | Prüfen ob ein Dialog läuft (z.B. für Overlap-Guards). |
| `Get Dialogue Subsystem` | Callable | `WorldContext` | `UMayDialogueSubsystem*` | Subsystem holen für Delegate-Binding. |

`WorldContext` wird in Blueprint automatisch vom Node-Kontext befüllt (kein manueller Pin).

---

## Signaturen

### Start Dialogue

```cpp
UFUNCTION(BlueprintCallable, Category = "MayDialogue",
    meta = (WorldContext = "WorldContext", DefaultToSelf = "Instigator"))
static UMayDialogueInstance* StartDialogue(
    UObject*           WorldContext,
    UMayDialogueAsset* Asset,
    AActor*            Instigator,
    AActor*            Target);
```

Liefert die erzeugte Instance oder `nullptr` bei Fehler (kein Asset, kein Entry, Instigator/Target ungültig). Startet einen laufenden Dialog neu — der alte wird abgebrochen.

---

### Stop Dialogue

```cpp
UFUNCTION(BlueprintCallable, Category = "MayDialogue")
static void StopDialogue(UMayDialogueInstance* Instance);
```

No-op bei `nullptr`. Bricht die angegebene Instance sauber ab.

---

### Stop All Dialogues

```cpp
UFUNCTION(BlueprintCallable, Category = "MayDialogue",
    meta = (WorldContext = "WorldContext"))
static void StopAllDialogues(UObject* WorldContext);
```

Bricht alle aktiven Instances ab. Typisch bei Level-Travel, Spielertod, Pause-Menü.

---

### Get Active Dialogue

```cpp
UFUNCTION(BlueprintCallable, Category = "MayDialogue",
    meta = (WorldContext = "WorldContext"))
static UMayDialogueInstance* GetActiveDialogue(UObject* WorldContext);
```

Liefert die (einzige) aktive Instance oder `nullptr`. Da das Subsystem nur einen Dialog gleichzeitig erlaubt, ist das praktisch immer eindeutig.

---

### Is Any Dialogue Active

```cpp
UFUNCTION(BlueprintPure, Category = "MayDialogue",
    meta = (WorldContext = "WorldContext"))
static bool IsAnyDialogueActive(UObject* WorldContext);
```

Schnell-Query. Kostet nichts — schlägt intern nur `Subsystem->IsAnyDialogueActive()` nach.

---

### Get Dialogue Subsystem

```cpp
UFUNCTION(BlueprintCallable, Category = "MayDialogue",
    meta = (WorldContext = "WorldContext"))
static UMayDialogueSubsystem* GetDialogueSubsystem(UObject* WorldContext);
```

Gibt das Subsystem zurück. Wenn du es öfter brauchst, in einer Blueprint-Variable cachen.

---

## Blueprint-Beispiele

### Start mit Fehler-Check

```text
[Start Dialogue]
  ├─ Asset:      DA_Merchant
  ├─ Instigator: Player Pawn
  └─ Target:     Merchant Actor
       │ Return Value
       ▼
[Is Valid]  → Branch
  ├─ True  → Weiter (Dialog läuft)
  └─ False → Print String "Dialog fehlgeschlagen"
```

> 📸 **Bild-Platzhalter:** `library-start-with-check-bp.png` — Vollständiger BP-Graph: Start Dialogue → Is Valid → Branch.
> *Setup:* Interaction-Blueprint. Nodes: `Start Dialogue` (Pins befüllt), `Is Valid` (Input: Return Value der Start-Node), `Branch`, True-Zweig leer, False-Zweig → `Print String`. Alle Verbindungen sichtbar.

### Alle Dialoge beim Spielertod stoppen

```text
[Event On Player Died]
    │
    ▼
[Stop All Dialogues]
```

### Subsystem für Delegate-Binding holen

```text
[Get Dialogue Subsystem] → (In Variable "Subsystem" speichern)
    │
    ▼
[Bind Event to On Any Dialogue Ended]  ──► Custom Event: Handle Ended
```

---

## Was die Library nicht kann

Die Library hat **keine Delegate-Properties**. Für `OnAnyDialogueStarted` / `OnAnyDialogueEnded` brauchst du das Subsystem direkt (via `Get Dialogue Subsystem`).

Auch nicht in der Library:
- Variable Get/Set → am Subsystem oder an der Instance direkt
- `CanStartDialogue` → am Subsystem direkt

## Siehe auch

- [API: Subsystem](api-subsystem.md) — vollständige Subsystem-API mit Delegates.
- [API: Delegates](api-delegates.md) — alle Delegate-Signaturen.
- [Runtime → Einen Dialog starten](../runtime/starting-dialogues.md) — geführter Walkthrough.
