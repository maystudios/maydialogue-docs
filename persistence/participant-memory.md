# Participant-Memory

`PersistentMemory` auf der `UMayDialogueParticipant`-Komponente ist ein **typed dynamic dictionary**, das Dialog-überlappende Variablen hält.

## Datenmodell

```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite, SaveGame, Category="Persistence")
FInstancedPropertyBag PersistentMemory;
```

`FInstancedPropertyBag` (aus dem `StructUtils`-Modul) speichert typisierte Key-Value-Paare. Jede Variable hat:

* Name (`FName`).
* Typ (Bool/Int/Float/String/GameplayTag).
* Wert.

## Setter

```cpp
void SetPersistentBool(FName Name, bool Value);
void SetPersistentInt(FName Name, int32 Value);
void SetPersistentFloat(FName Name, float Value);
void SetPersistentString(FName Name, const FString& Value);
void SetPersistentTag(FName Name, const FGameplayTag& Value);
```

Alle broadcasten anschließend `OnVariableChanged` am Participant.

## Getter mit Default

```cpp
bool GetPersistentBool(FName Name, bool DefaultValue = false) const;
int32 GetPersistentInt(FName Name, int32 DefaultValue = 0) const;
float GetPersistentFloat(FName Name, float DefaultValue = 0.0f) const;
FString GetPersistentString(FName Name, const FString& DefaultValue = TEXT("")) const;
FGameplayTag GetPersistentTag(FName Name, const FGameplayTag& DefaultValue = FGameplayTag()) const;
```

Wenn die Variable nicht existiert: liefert den Default.

## String-API für Bridge

```cpp
bool GetPersistentVariableAsString(FName Name, EMayDialogueVariableType Type, FString& OutValue) const;
bool SetPersistentVariableFromString(FName Name, EMayDialogueVariableType Type, const FString& ValueAsString);
```

Genutzt vom [Read/Write-API](../runtime/read-write-api.md) über die Bridge.

## Typische Nutzung

### In-Dialog

Durch SideEffect-SetVariable-Nodes oder Action-SetVariable-Nodes – alles mit `Scope = Participant`.

### Aus Code

```cpp
void UQuestSystem::MarkGuardMet(AGuardActor* Guard)
{
    auto* Part = Guard->FindComponentByClass<UMayDialogueParticipant>();
    if (!Part) return;

    Part->SetPersistentBool("HasMet", true);
    Part->SetPersistentInt("MeetingCount", Part->GetPersistentInt("MeetingCount") + 1);
}
```

## Events

Die Komponente broadcastet:

* `OnVariableChanged(Name, Scope=Participant, Type, NewValueAsString)`.

Externe Systeme können hören:

```cpp
Part->OnVariableChanged.AddDynamic(this, &ThisClass::HandleVarChange);

void AMyListener::HandleVarChange(
    FName VarName,
    EMayDialogueVariableScope Scope,
    EMayDialogueVariableType Type,
    FString NewValueAsString)
{
    if (VarName == "IsAngry" && NewValueAsString == "true")
    {
        TriggerCombat();
    }
}
```

## Lebenszyklus

* **Erstellung**: Beim Actor-Spawn wird die Komponente initialisiert, PropertyBag ist leer (oder enthält Save-geladene Werte).
* **Runtime**: Getter/Setter aktiv.
* **Destruction**: Beim Actor-Destroy wird die Komponente zerstört; PersistentMemory geht verloren, wenn nicht zuvor gespeichert.

## Anmerkungen

* Die PropertyBag kennt nur die 5 Grund-Typen. Komplexere Strukturen: eigenes UPROPERTY am Participant-Blueprint-Subclass + `SaveGame`-Flag.
* Beim Schreiben eines nicht-existierenden Namens wird die Variable automatisch angelegt.
* Typ-Mismatch (z.B. `GetPersistentBool` auf einer Int-Variable) liefert den Default-Wert, nicht den konvertierten Wert.
