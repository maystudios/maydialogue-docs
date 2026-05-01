# Choice

Eine Choice ist eine Antwort-Option auf einem [PlayerChoice-Node](../core/player-choice.md). Sie definiert den Text, den der Spieler sieht, die Bedingungen für ihre Verfügbarkeit und Aktionen, die bei Auswahl ausgeführt werden.

## Wann setze ich ihn ein?

- Immer zusammen mit einem PlayerChoice-Node — jede Option ist eine Choice.
- Wenn eine Antwort-Option nur für Spieler mit bestimmten Tags/Attributen verfügbar sein soll (Requirements auf der Choice).
- Wenn bei Auswahl einer Option sofort eine Aktion ausgelöst werden soll, ohne einen eigenen Action-Node (SideEffects auf der Choice).
- Mit `ChoiceTags`, wenn externe Systeme (Analytics, Achievement, UI-Styling) den Charakter der Antwort kennen sollen.
- Mit `UnavailableReason`, um dem Spieler zu erklären, warum eine Option ausgegraut ist.

## Properties

| Property | Typ | Standard | Bedeutung |
| --- | --- | --- | --- |
| `ChoiceText` | `FText` | leer | Der Text des Buttons, den der Spieler sieht. |
| `ChoiceTags` | `FGameplayTagContainer` | leer | Metadaten-Tags, z.B. `Choice.Aggressive`, `Choice.Peaceful`. Für UI-Styling und externe Systeme. |
| `Requirements` | Array `UMayDialogueRequirement*` | leer | Bestimmen die Verfügbarkeit der Choice (Passed / FailedButVisible / FailedAndHidden). |
| `SideEffects` | Array `UMayDialogueSideEffect*` | leer | Werden bei Auswahl dieser Choice ausgeführt, bevor zum nächsten Node gesprungen wird. |
| `UnavailableReason` | `FText` | leer | Tooltip-Text, wenn die Choice `FailedButVisible` ist. |
| `TargetNodeGuid` | `FGuid` | (Compiler) | Zeigt auf den nächsten Node (aus dem Output-Pin). Wird vom Compiler gesetzt. |

> 📸 **Bild-Platzhalter:** `choice-pill-on-playerchoice.png` — PlayerChoice-Node mit drei Choice-Pills und aufgeklappter Choice im Details-Panel.
> *Setup:* PlayerChoice-Node mit drei Choice-Pills im Body: `"Ein Freund des Königs."` (keine Requirements), `"Das Passwort kennen."` (Requirement-Pill `HasTag Story.HeardPassword`), `"Mit Geld bestechen."` (Requirement-Pill `CheckAttribute Gold >= 100`, ausgegraut im Preview). Details-Panel zeigt Choice 2: `ChoiceText`, `UnavailableReason = "Du hast nicht genug Gold."`, Requirements-Array.

> 📸 **Bild-Platzhalter:** `choice-details-panel.png` — Details-Panel einer einzelnen Choice.
> *Setup:* Choice-Eintrag aufklappen. Sichtbar: `ChoiceText = "Mit Geld bestechen."`, `ChoiceTags = (Choice.Bribe)`, `UnavailableReason = "Du hast nicht genug Gold."`, `Requirements` (1 Eintrag: CheckAttribute Gold >= 100), `SideEffects` (1 Eintrag: ApplyEffect GE_RemoveGold).

> 📸 **Bild-Platzhalter:** `choice-with-sideeffect-pill.png` — Choice mit SideEffect-Pill sichtbar im Node-Body.
> *Setup:* PlayerChoice-Node. Choice-Pill `"Angriff!"` aufgeklappt. Im Body dieser Choice sichtbar: SideEffect-Pill `ApplyEffect GE_Adrenaline`. Zeigt Eltern-Node (PlayerChoice) → Choice-Pill → SideEffect-Pill (verschachtelt).

## Mini-Beispiel

```text
[PlayerChoice: "Was willst du tun?"]
  │
  ├─ Choice 0 "Angriff!"
  │    ChoiceTags: Choice.Aggressive
  │    SideEffect: ApplyEffect GE_Adrenaline
  │    ──► [SayLine: "Dann kämpfen wir!"] ──► [Exit: Failed]
  │
  ├─ Choice 1 "Verhandeln."
  │    Requirement: HasTag Story.Peaceful (FailedButVisible)
  │    UnavailableReason: "Du musst friedlich bekannt sein."
  │    ──► [SayLine: "Gut, lass uns reden."] ──► [Exit: Completed]
  │
  └─ Choice 2 "Fliehen."
       Requirement: CheckAttribute Stamina > 20 (FailedButVisible)
       UnavailableReason: "Du bist zu erschöpft."
       ──► [SayLine: "Du rennst davon!"] ──► [Exit: Completed]
```

> 📸 **Bild-Platzhalter:** `choice-example-graph.png` — Vollständiger PlayerChoice-Graph mit drei Choices.
> *Setup:* `Entry` → `SayLine "Was willst du tun?"` → `PlayerChoice`. Drei Choice-Pills: `"Angriff!"` (SideEffect-Pill), `"Verhandeln."` (Requirement-Pill, ausgegraut), `"Fliehen."` (Requirement-Pill). Drei Output-Pfade jeweils zu SayLine → Exit.

## Häufige Fallstricke

- **Requirements werden bei Auswahl erneut geprüft**: Wenn sich zwischen Anzeige und Klick eine Variable ändert, kann die Choice bei SelectChoice abgelehnt werden, auch wenn sie beim Anzeigen verfügbar war.
- **`UnavailableReason` leer bei `FailedButVisible`**: Der Spieler sieht die ausgegraut Option ohne Erklärung. Fülle `UnavailableReason` immer aus, wenn du `FailResult = FailedButVisible` nutzt.

## Erweitern

{% hint style="success" %}
**Eigene Choice-Logik in Blueprint:**

Leite eine Blueprint-Subklasse von `UMayDialogueChoice` ab und überschreibe das Event `OnChoiceSelected`. So kannst du bei jeder Auswahl eigene Logik ausführen — ohne den Dialog-Graph zu verändern.

Details: [Extension → Custom Choices](../../extension/custom-choices.md).
{% endhint %}
