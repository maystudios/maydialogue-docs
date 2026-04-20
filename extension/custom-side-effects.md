# Eigene SideEffects

SideEffects sind **Inline-Aktionen** am Eltern-Node. Für projekt-spezifische Aktionen (Quest-Update, Inventory-Add, Achievement-Grant) erstellst du eigene Subklassen.

## Blueprint-Variante

1. Rechtsklick → Blueprint Class → `MayDialogueSideEffect` als Parent.
2. Benennen, z.B. `BP_SE_GrantAchievement`.
3. Event Graph → **Override** → `Execute Side Effect`.

```
Event Execute Side Effect (Context):
  - Get Achievement Subsystem from World
  - Grant Achievement with ID = AchievementID
```

Properties im Blueprint:

* `AchievementID: FName` (Instance Editable = true).

## C++-Variante

```cpp
UCLASS(BlueprintType, EditInlineNew, meta=(DisplayName="Grant Achievement"))
class MYGAME_API UMySE_GrantAchievement : public UMayDialogueSideEffect
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, Category="Achievements")
    FName AchievementID;

    virtual void ExecuteSideEffect(const FMayDialogueContext& Context) override;
    virtual FText GetDisplayDescription() const override;
};
```

```cpp
void UMySE_GrantAchievement::ExecuteSideEffect(const FMayDialogueContext& Context)
{
    if (AchievementID == NAME_None) return;

    if (auto* Ach = UAchievementSubsystem::Get(Context.GetWorld()))
    {
        Ach->Grant(AchievementID);
    }
}

FText UMySE_GrantAchievement::GetDisplayDescription() const
{
    return FText::Format(
        NSLOCTEXT("MayDialogue", "GrantAchievementDesc", "Grant: {0}"),
        FText::FromName(AchievementID)
    );
}
```

## Server/Client-Split

SideEffect hat **zwei** virtuelle Methoden:

```cpp
virtual void ExecuteSideEffect(const FMayDialogueContext& Context);      // Gameplay-Logik
virtual void ExecuteClientSideEffect(const FMayDialogueContext& Context); // Rein cosmetic (Sounds, Particles)
```

Für Multiplayer-Szenarien:

* `ExecuteSideEffect` läuft **nur auf dem Server** (Gameplay-Änderung).
* `ExecuteClientSideEffect` läuft **auf jedem Client** (visuelle Bestätigung).

Für Singleplayer egal — beide werden aufgerufen.

## Composable SideEffects

Auf einem Node kannst du mehrere SideEffects hintereinander legen:

```
SayLine "Ich gebe dir Gold"
  SideEffects:
    + ApplyEffect: GE_GoldGain
    + GrantAchievement: First_Reward_Received
    + Inventory_Add: Item_GoldCoin x 10
    + PlaySound: S_CoinJingle
```

Reihenfolge = Array-Order.

## Design-Tipps

* **Ein SideEffect pro Aktion**. Nicht ein Super-SideEffect mit 10 Flags.
* **Description dynamisch**: via `GetDisplayDescription()` mit aktuellem Property-Wert. Das macht Graph-Pills informativ.
* **Idempotenz, wenn möglich**: Falls der Dialog rückgängig oder zweimal gespielt wird, sollte dein SideEffect nicht kaputt gehen.
* **Keine User-Input in SideEffects**. Sie laufen inline; kein Warten, keine Choice-Präsentation.

## Typische projekt-spezifische SideEffects

* `SE_GrantAchievement(ID)`.
* `SE_Inventory_Add(ItemID, Count)`.
* `SE_Inventory_Remove(ItemID, Count)`.
* `SE_QuestAdvance(QuestID, StageDelta)`.
* `SE_NPCReputation(FactionTag, Delta)`.
* `SE_UnlockDoor(DoorActor)`.
* `SE_NotifyAnalytics(EventName, Payload)`.

Alle als einzelne Blueprint- oder C++-Klassen, sauber separiert.
