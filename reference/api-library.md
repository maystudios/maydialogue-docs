# `UMayDialogueLibrary` (Referenz)

Blueprint-Helper-Klasse (`UBlueprintFunctionLibrary`). Alle Methoden sind `static`, `BlueprintCallable` oder `BlueprintPure`. Die Library ist ein **reiner Convenience-Layer** über dem [Subsystem](api-subsystem.md) – keine eigene Logik.

* **Header**: `Source/MayDialogue/Runtime/MayDialogueLibrary.h`
* **Modul**: `MayDialogue`
* **Blueprint-Category**: `Dialogue`

## Methoden im Überblick

| Signatur | Art | Kurzbeschreibung |
| --- | --- | --- |
| `StartDialogue(WC, Asset, Instigator, Target)` | `BlueprintCallable` | Startet ein Gespräch, liefert die Instance oder `nullptr`. |
| `StopDialogue(Instance)` | `BlueprintCallable` | Bricht eine bestimmte Instance ab. |
| `StopAllDialogues(WC)` | `BlueprintCallable` | Bricht alle aktiven Instances ab. |
| `GetActiveDialogue(WC)` | `BlueprintPure` | Liefert die aktive Instance oder `nullptr`. |
| `IsAnyDialogueActive(WC)` | `BlueprintPure` | `true`, wenn irgendein Dialog läuft. |
| `GetDialogueSubsystem(WC)` | `BlueprintPure` | Liefert das Subsystem-Objekt. |

`WC` steht für `UObject* WorldContext` – in Blueprint wird der Pin automatisch vom Node-Self-Kontext befüllt.

## Signaturen im Detail

### StartDialogue

```cpp
UFUNCTION(BlueprintCallable, Category="Dialogue",
          meta=(WorldContext="WorldContext", DisplayName="Start Dialogue"))
static UMayDialogueInstance* StartDialogue(
    UObject*            WorldContext,
    UMayDialogueAsset*  Asset,
    AActor*             Instigator,
    AActor*             Target);
```

Delegiert an `UMayDialogueSubsystem::StartDialogue`. Rückgabewert ist die erzeugte Instance oder `nullptr` bei Failure (kein Asset, kein Entry, bereits aktiver Dialog, der nicht abgebrochen werden darf).

### StopDialogue

```cpp
UFUNCTION(BlueprintCallable, Category="Dialogue")
static void StopDialogue(UMayDialogueInstance* Instance);
```

No-op bei `Instance == nullptr`. Sonst `Instance->AbortDialogue()`.

### StopAllDialogues

```cpp
UFUNCTION(BlueprintCallable, Category="Dialogue",
          meta=(WorldContext="WorldContext"))
static void StopAllDialogues(UObject* WorldContext);
```

Typische Nutzung: Level-Travel, Player-Death, Pause-Menü.

### GetActiveDialogue

```cpp
UFUNCTION(BlueprintPure, Category="Dialogue",
          meta=(WorldContext="WorldContext"))
static UMayDialogueInstance* GetActiveDialogue(UObject* WorldContext);
```

Liefert die „neueste" aktive Instance. Da das Subsystem Single-Active erzwingt, ist das praktisch die einzige.

### IsAnyDialogueActive

```cpp
UFUNCTION(BlueprintPure, Category="Dialogue",
          meta=(WorldContext="WorldContext"))
static bool IsAnyDialogueActive(UObject* WorldContext);
```

### GetDialogueSubsystem

```cpp
UFUNCTION(BlueprintPure, Category="Dialogue",
          meta=(WorldContext="WorldContext"))
static UMayDialogueSubsystem* GetDialogueSubsystem(UObject* WorldContext);
```

Wenn du das Subsystem öfter brauchst (Delegates binden, mehrere Calls), cache die Referenz in einer Variable.

## Blueprint-Beispiele

### Mini-Workflow: Start → Hat gestartet?

```
[Start Dialogue]
  ├ Asset:      DA_Villager_Intro
  ├ Instigator: Player Pawn
  └ Target:     Self
       │
       ▼
[Branch: Return Value != nullptr]
  ├─ True  → Dialog läuft
  └─ False → Log-Warn: „Dialog konnte nicht starten"
```

### Aktiven Dialog lesen

```
[Is Any Dialogue Active]
   │ (bool)
   ▼
[Branch]
  └─ True → [Get Active Dialogue] → (UMayDialogueInstance)
```

### Alle Dialoge bei Spielertod killen

```
[Event On Player Died]
   │
   ▼
[Stop All Dialogues]
```

## C++-Nutzung

Obwohl die Library für Blueprint gedacht ist, kannst du sie auch aus C++ aufrufen:

```cpp
UMayDialogueInstance* Inst = UMayDialogueLibrary::StartDialogue(
    this, DialogueAsset, Player, NPC);

if (UMayDialogueLibrary::IsAnyDialogueActive(this))
{
    UMayDialogueLibrary::StopAllDialogues(this);
}
```

Für längere Code-Pfade ist der direkte Subsystem-Weg jedoch idiomatischer:

```cpp
UMayDialogueSubsystem* Sub = UMayDialogueSubsystem::Get(GetWorld());
if (Sub)
{
    Sub->StartDialogue(DialogueAsset, Player, NPC);
}
```

## Was der Library fehlt (bewusst)

Die Library stellt **bewusst** keine Delegates bereit. Wenn du auf `OnDialogueStarted` / `OnDialogueEnded` lauschen willst, musst du das [Subsystem](api-subsystem.md) direkt holen – dort hängen die Delegates. Das vermeidet versehentliche Mehrfach-Bindings auf einer stateless Library.

Auch nicht in der Library:

* Variable-Read/Write → auf der [`UMayDialogueInstance`](../runtime/read-write-api.md).
* Dialogue-Validation → auf dem Asset.
* SaveGame-Serialisierung → [`UMayDialogueSaveHelper`](../persistence/quicksave-helper.md).

## Thread-Safety

Alle Methoden sind **Game-Thread-only**. Das Subsystem dahinter erwartet `GWorld`-Kontext.

## Siehe auch

* [`UMayDialogueSubsystem`](api-subsystem.md)
* [Delegates & Events](api-delegates.md)
* [Runtime → Einen Dialog starten](../runtime/starting-dialogues.md) – narrativer Startpunkt.
