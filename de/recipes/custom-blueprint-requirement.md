---
description: Eigenen Requirement-Typ in Blueprint bauen – erscheint automatisch in der Node-Palette.
---

# Eigenen Requirement in Blueprint bauen

## Szenario

Du brauchst eine Bedingung, die MayDialogue noch nicht liefert: *„Spieler trägt Item X im Inventar"*. Statt dein Inventar-System in einen GAS-Tag zu übersetzen, baust du direkt einen eigenen Blueprint-Requirement. Er erscheint automatisch in der Node-Palette und verhält sich wie jedes andere Requirement.

## Was du lernst

- Blueprint-Subklasse von `UMayDialogueRequirement` anlegen.
- `IsRequirementSatisfied()`-Funktion in Blueprint implementieren.
- `Passed`, `FailedButVisible`, `FailedAndHidden` als Return-Wert nutzen.
- Eigene Properties für den Editor konfigurieren (Inventar-Item-Name).
- Den neuen Requirement-Typ im Dialog-Asset nutzen.

## Voraussetzungen

- [Verzweigungen mit Bedingungen](branching-conditions.md) abgeschlossen.
- Inventar-System mit einer zugänglichen `HasItem(FName)`-Funktion.

## Was du baust

Blueprint `BPReq_HasInventoryItem` – ein Requirement, das prüft ob der Spieler ein bestimmtes Item im Inventar hat. Konfigurierbar über ein `ItemName`-Property im Details-Panel.

> 📸 **Bild-Platzhalter:** `custom-blueprint-requirement-palette.png` — Node-Palette im Dialog-Editor mit dem neuen Requirement-Typ.
> *Setup:* PlayerChoice-Node im Asset-Editor, Requirement-Dropdown geöffnet. In der Liste sichtbar: Standard-Requirements (HasTag, CheckAttribute, ...) und darunter der neue Eintrag `Has Inventory Item (Custom)`.

## Schritt-für-Schritt

### 1. Blueprint-Klasse anlegen

Content Browser → **Rechtsklick → Blueprint Class**. In der Klassen-Auswahl nach `MayDialogueRequirement` suchen und auswählen.

Name: `BPReq_HasInventoryItem`.

### 2. ItemName-Property hinzufügen

Im Blueprint-Editor: **Variables → Add Variable**:
- `Variable Name`: `ItemName`
- `Variable Type`: `Name`
- `Instance Editable`: `true` (damit es im Dialog-Editor einstellbar ist)
- `Expose on Spawn`: `true`

> 📸 **Bild-Platzhalter:** `custom-blueprint-requirement-variable.png` — Blueprint-Editor mit ItemName-Variable.
> *Setup:* `BPReq_HasInventoryItem` Blueprint geöffnet. Variables-Panel links: `ItemName (Name, Instance Editable)` sichtbar. Details-Panel rechts zeigt Checkbox `Instance Editable = true`.

### 3. IsRequirementSatisfied-Funktion überschreiben

Im Event-Graph: **Override → Is Requirement Satisfied**. Die Funktion bekommt den `Context` (ein `FMayDialogueContext`) und soll einen `EMayDialogueRequirementResult` zurückgeben. Den Instigator liest du aus dem Context (`Context → Instigator`).

```text
[Is Requirement Satisfied (Context)]
   │
   ▼
[Context → Instigator] → [Get Inventory Component]
   │
   ▼
[Has Item? (ItemName)]
   ├─ True  → [Return: Passed]
   └─ False → [Return: Get Fail Result (Self)]
```

Mit `Get Fail Result (Self)` im Fehlerfall respektierst du das `FailResult`-Property, das du pro Instanz im Details-Panel setzt — derselbe Blueprint kann so an einer Choice ausgegraut und an einer anderen versteckt rendern.

> 📸 **Bild-Platzhalter:** `custom-blueprint-requirement-graph.png` — IsRequirementSatisfied-Implementierung im Blueprint-Graph.
> *Setup:* `BPReq_HasInventoryItem` Event-Graph. Override `Is Requirement Satisfied` ausgeklappt. Nodes: `Context` (Struct) → `Break` / `Instigator`-Pin → `Get Component by Class (InventoryComponent)` → `Has Item (ItemName Variable)` → Branch → zwei Return-Nodes: `Return Passed` (grün) und `Return Get Fail Result` (gelb).

### 4. Requirement im Dialog-Asset nutzen

In jedem Requirement-Dropdown (Branch, Choice, RandomLine-Entry) erscheint `BPReq_HasInventoryItem` jetzt automatisch als Option.

Eine Choice (oder eine Branch-Condition) auswählen → **Requirements → Add → Has Inventory Item** → `ItemName`-Property im Details-Panel ausfüllen:

| Property | Wert |
|----------|------|
| `ItemName` | `Key_RedDoor` |
| `FailResult` | `FailedAndHidden` |

> 📸 **Bild-Platzhalter:** `custom-blueprint-requirement-in-use.png` — Details-Panel einer Choice mit dem neuen Requirement.
> *Setup:* Dialog-Asset geöffnet, Choice ausgewählt. Details: Requirement-Liste zeigt `BPReq_HasInventoryItem`, darunter Property `ItemName = "Key_RedDoor"`, `FailResult = FailedAndHidden`.

### 5. Compile und testen

Im PIE: Dialog starten ohne das Item → Choice versteckt. Item ins Inventar legen, Dialog neu starten → Choice sichtbar.

## Rückgabe-Werte

| Wert | Bedeutung |
|------|-----------|
| `Passed` | Bedingung erfüllt |
| `FailedButVisible` | Nicht erfüllt, aber Option sichtbar + ausgegraut |
| `FailedAndHidden` | Nicht erfüllt, Option vollständig versteckt |

Gib bevorzugt `Get Fail Result (Self)` zurück und lass das Pro-Instanz-`FailResult`-Property über sichtbar vs. versteckt entscheiden — so steuern Designer es pro Choice, ohne den Blueprint zu ändern. Wähle `FailedButVisible` wenn der Spieler wissen soll, dass die Option existiert (z.B. *„Du brauchst einen Schlüssel"*). Wähle `FailedAndHidden` für versteckte Pfade.

## C++-Variante

Für mehr Kontrolle und Performance: C++-Subklasse. Überschreibe `IsRequirementSatisfied_Implementation` (die `_Implementation`-Hälfte des `BlueprintNativeEvent` — markiere sie **nicht** erneut als `UFUNCTION`).

```cpp
UCLASS()
class UMayDlgRequirement_HasInventoryItem : public UMayDialogueRequirement
{
    GENERATED_BODY()
public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Requirement")
    FName ItemName;

    virtual EMayDialogueRequirementResult IsRequirementSatisfied_Implementation(
        const FMayDialogueContext& Context) const override
    {
        if (UInventoryComponent* Inv = Context.Instigator
            ? Context.Instigator->FindComponentByClass<UInventoryComponent>()
            : nullptr)
        {
            return Inv->HasItem(ItemName)
                ? EMayDialogueRequirementResult::Passed
                : GetFailResult();
        }
        return GetFailResult();
    }
};
```

`GetFailResult()` kommt aus der Basisklasse und resolved Hidden vs. Visible aus dem Editor-konfigurierbaren `FailResult`-Property.

## Variation / Weiter gehen

- Mehrere Properties: `MinQuantity` für „mindestens 3 Tränke".
- Requirement kombinieren: zusammen mit HasTag an derselben Choice (AND-Logik).
- Eigene SideEffects bauen: dieselbe Basis-Klasse, andere Funktion → [Erweiterung → Custom SideEffects](../extension/custom-side-effects.md).

## Troubleshooting

**Requirement taucht nicht in der Palette auf.**
Blueprint muss von `UMayDialogueRequirement` erben (direkt oder indirekt). Klassen-Selektion beim Erstellen prüfen. Editor neu starten nach dem ersten Anlegen.

**`ItemName`-Property erscheint nicht im Details-Panel.**
`Instance Editable` am Variable nicht aktiviert. Im Blueprint-Editor: Variable auswählen → Details → Checkbox `Instance Editable = true`.

**IsRequirementSatisfied gibt immer Passed zurück.**
Inventar-Komponente auf dem Instigator nicht gefunden (`Get Component by Class` returned null). Prüfe ob der Spieler-Pawn die Komponente hat und ob `Context.Instigator` der Spieler ist (nicht der NPC).
