# Blueprint-Library

`UMayDialogueLibrary` ist eine `UBlueprintFunctionLibrary` mit statischen Bequemlichkeits-Methoden. Sie wrappt das Subsystem.

## Alle Methoden

```cpp
UFUNCTION(BlueprintCallable, meta=(WorldContext="WorldContext"))
static UMayDialogueInstance* StartDialogue(
    UObject*            WorldContext,
    UMayDialogueAsset*  Asset,
    AActor*             Instigator,
    AActor*             Target);

UFUNCTION(BlueprintCallable)
static void StopDialogue(UMayDialogueInstance* Instance);

UFUNCTION(BlueprintCallable, meta=(WorldContext="WorldContext"))
static void StopAllDialogues(UObject* WorldContext);

UFUNCTION(BlueprintPure, meta=(WorldContext="WorldContext"))
static UMayDialogueInstance* GetActiveDialogue(UObject* WorldContext);

UFUNCTION(BlueprintPure, meta=(WorldContext="WorldContext"))
static bool IsAnyDialogueActive(UObject* WorldContext);

UFUNCTION(BlueprintPure, meta=(WorldContext="WorldContext"))
static UMayDialogueSubsystem* GetDialogueSubsystem(UObject* WorldContext);
```

## Wann Library, wann Subsystem direkt?

| Szenario | Empfehlung |
| --- | --- |
| Blueprint-Quick-Start | Library (weniger Boilerplate) |
| Wiederholte Subsystem-Calls im selben Scope | Subsystem-Referenz cachen |
| Multi-Event-Binding | Subsystem direkt (Delegates sind auf dem Subsystem) |
| System-Code, der sowieso das Subsystem braucht | Subsystem direkt |

## Blueprint-Beispiele

### Dialog starten

```
[MayDialogueLibrary :: Start Dialogue]
  ├ World Context: Self
  ├ Asset:         (Dialog-Asset)
  ├ Instigator:    Player Pawn
  └ Target:        NPC Actor
```

### Aktiven Dialog abfragen

```
[MayDialogueLibrary :: Get Active Dialogue] → (Instance-Referenz)
```

### Alle Dialoge stoppen bei Tod des Spielers

```
[Event On Player Died]
  │
  ▼
[MayDialogueLibrary :: Stop All Dialogues]
  └ World Context: Self
```

## Anmerkungen

* Die Library ist ein **reiner Convenience-Layer**. Keine eigene Logik, kein eigener State – sie delegiert alles an das Subsystem.
* Wenn du Subsystem-Delegates brauchst (z.B. `OnAnyDialogueStarted`), musst du das Subsystem direkt referenzieren – die Library hat keine Delegate-Properties.
