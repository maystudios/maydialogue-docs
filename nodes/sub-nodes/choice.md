# Choice

Eine **Choice** ist eine Antwort-Option auf einem PlayerChoice-Node.

## Basis-Klasse

`UMayDialogueChoice` (EditInlineNew, BlueprintType, Blueprintable):

| Property | Typ | Zweck |
| --- | --- | --- |
| `ChoiceText` | `FText` | Der Text, den der Spieler als Button sieht. |
| `ChoiceTags` | `FGameplayTagContainer` | Metadaten für externe Systeme (Achievements, Analytics). |
| `Requirements` | `TArray<UMayDialogueRequirement*>` | Availability-Logik. |
| `SideEffects` | `TArray<UMayDialogueSideEffect*>` | Werden beim SelectChoice ausgeführt. |
| `UnavailableReason` | `FText` | Tooltip bei FailedButVisible. |
| `TargetNodeGuid` | `FGuid` (Compiler) | Zeigt auf den Next-Node (aus dem Output-Pin). |

## Verhalten

Bei jedem PlayerChoice-Execute:

1. `EvaluateRequirements(Context)` – kombiniert alle Requirements.
2. `BuildChoiceEntry(Context, Index)` – erzeugt `FMayDialogueChoiceEntry` mit Text, Tags, Availability.
3. FailedAndHidden werden entfernt.
4. Rest wird der UI via `OnChoicesPresented` geliefert.

Bei `SelectChoice(Index)`:

1. Re-Evaluate Requirements.
2. `OnChoiceSelected(Context)` (BlueprintNativeEvent) – Basis-Implementation führt SideEffects aus.
3. Sprung zum `TargetNodeGuid`.

## Typisches Pattern

```
[PlayerChoice: "Was willst du tun?"]
  │
  ├─ Choice 0: "Kampf!"       (ChoiceTags: Choice.Aggressive)
  │   SideEffect: ApplyEffect GE_Adrenaline
  │
  ├─ Choice 1: "Verhandeln."  (Req: HasTag Story.Peaceful)
  │
  └─ Choice 2: "Fliehen."     (Req: CheckAttribute Stamina > 20,
                               FailureResult=FailedButVisible)
      UnavailableReason: "Du bist zu erschöpft."
```

## Anmerkungen

* **ChoiceTags** sind die Hook-Points für externe Systeme. Ein Analytics-Logger auf `OnChoiceMade` liest die Tags und kann z.B. *„Spieler hat aggressive Choice gewählt"* tracken, ohne den Dialog-Code zu kennen.
* `UnavailableReason` ist ein **UX-Hilfsmittel** – zeigt dem Spieler, warum eine Option nicht wählbar ist.
* Eigene Choice-Subklassen (Blueprint) können den `OnChoiceSelected`-Event überschreiben und spezifische Logik injizieren.
