# QuickSave-Helper

Für Projekte, die **kein eigenes SaveGame-System** haben, liefert MayDialogue einen minimalen Helfer: `UMayDialogueSaveHelper`.

{% hint style="info" %}
**Status**: Als Backlog-Item 18 geplant. Die API ist definiert, die Implementierung wird gerade finalisiert. Die hier dokumentierten Funktionen sind der Design-Vertrag.
{% endhint %}

## API

```cpp
UCLASS()
class MAYDIALOGUE_API UMayDialogueSaveHelper : public UBlueprintFunctionLibrary
{
    UFUNCTION(BlueprintCallable, meta=(WorldContext="WorldContext"))
    static bool QuickSaveToSlot(UObject* WorldContext, const FString& SlotName, int32 UserIndex = 0);

    UFUNCTION(BlueprintCallable, meta=(WorldContext="WorldContext"))
    static bool QuickLoadFromSlot(UObject* WorldContext, const FString& SlotName, int32 UserIndex = 0);

    UFUNCTION(BlueprintCallable)
    static bool DeleteSlot(const FString& SlotName, int32 UserIndex = 0);

    UFUNCTION(BlueprintPure)
    static bool DoesSlotExist(const FString& SlotName, int32 UserIndex = 0);
};
```

## QuickSaveToSlot

1. Iteriert alle `UMayDialogueParticipant`-Komponenten in der Welt.
2. Extrahiert ihre `PersistentMemory` und den `ParticipantTag`.
3. Erzeugt ein `UMayDialogueSaveGame`-Objekt, füllt `ParticipantMemory` (TMap<FName, FInstancedPropertyBag>).
4. `UGameplayStatics::SaveGameToSlot(SaveObject, SlotName, UserIndex)`.

Liefert `true`, wenn erfolgreich.

## QuickLoadFromSlot

Inverse:

1. `UGameplayStatics::LoadGameFromSlot(SlotName, UserIndex)`.
2. Cast zu `UMayDialogueSaveGame`.
3. Iteriert `ParticipantMemory`.
4. Sucht zur Laufzeit Participants mit passendem Tag und schreibt `PersistentMemory`.

## Beispiel

```
// Beim Level-Verlassen
MayDialogueSaveHelper::QuickSaveToSlot(Self, "AutoSave", 0);

// Beim Level-Betreten
MayDialogueSaveHelper::QuickLoadFromSlot(Self, "AutoSave", 0);
```

## Einschränkungen

* **Nur MayDialogue-Daten**. Spielwelt-State, Inventar, Position etc. sind nicht Teil des QuickSave.
* **Keine Sicherheitsmerkmale**. Kein Versioning, keine Integrity-Checks.
* **Keine Delta-Saves**. Jeder Save schreibt alles neu.

## Wann Helper, wann eigenes System?

| Szenario | Empfehlung |
| --- | --- |
| Game-Jam-Prototyp | QuickSave-Helper. |
| Standard-Indie-Projekt | QuickSave-Helper als Start, eigenes System wenn nötig. |
| Produktions-Projekt mit komplexem State | Eigenes SaveGame-System, `PersistentMemory` per `ArIsSaveGame` einbetten. |

## Global Memory

`UMayDialogueSaveGame` hat zusätzlich `GlobalMemory` (`FInstancedPropertyBag`), das du für Projekt-weite Flags nutzen kannst. Vom QuickSaveHelper standardmäßig **nicht** befüllt – du musst GlobalMemory-Setter/Getter selbst aufrufen, falls du das Feld nutzen willst.

## Anmerkungen

* Slot-Namen sollten `[a-zA-Z0-9_-]` enthalten – sichere Dateinamen.
* `DoesSlotExist` prüft vor Load, um Load-Fehler zu vermeiden.
