# SideEffect

Ein SideEffect ist eine Inline-Aktion, die ausgeführt wird, sobald der Eltern-Node betreten wird — oder, im Fall einer Choice, wenn die Option gewählt wird. SideEffects leben als Pills im Node-Body und halten den Graphen sauber, ohne eine eigene Node-Box zu brauchen.

## Wann setze ich ihn ein?

- Wenn beim Betreten einer SayLine nebenbei eine Variable gesetzt oder ein Tag vergeben werden soll.
- Wenn bei Auswahl einer Choice eine Gameplay-Aktion ausgelöst werden soll (z.B. GAS-Effect anwenden).
- Für zwei bis drei gleichzeitige Inline-Aktionen an derselben Stelle — bei mehr lohnt sich ein eigener Action-Node.
- Auf dem Entry-Node, um den Dialog-Zustand beim Start zu initialisieren.
- Auf dem Exit-Node, um eine letzte Aktion vor dem Ende auszuführen.

## Properties (Basis-Klasse)

| Property | Typ | Standard | Bedeutung |
| --- | --- | --- | --- |
| `Description` | `FText` | leer | Editor-Label der Pill im Graph. Wird auch im Details-Panel angezeigt. |

## Standard-SideEffects (MayDialogueGAS-Modul)

| Klasse | Aktion | Wichtige Properties |
| --- | --- | --- |
| `UMayDlgSideEffect_AddTag` | Gameplay-Tag hinzufügen | `TagToAdd`, `bAddToInstigator` |
| `UMayDlgSideEffect_RemoveTag` | Gameplay-Tag entfernen | `TagToRemove`, `bRemoveFromInstigator` |
| `UMayDlgSideEffect_ApplyEffect` | GAS-GameplayEffect anwenden | `EffectClass`, `EffectLevel`, `bApplyToInstigator` |
| `UMayDlgSideEffect_TriggerCue` | GameplayCue feuern | `CueTag`, `bTriggerOnInstigator` |

Details: [GAS → Aktionen](../../gas/actions.md).

> 📸 **Bild-Platzhalter:** `sideeffect-pills-on-sayline.png` — SayLine-Node mit drei SideEffect-Pills im Body.
> *Setup:* SayLine-Node `"Hier ist dein Dank, Held."` auswählen. Im Node-Body drei SideEffect-Pills sichtbar: `ApplyEffect: GE_ReputationUp Lv.2`, `AddTag: Story.Guard.Grateful`, `TriggerCue: GameplayCue.UI.RewardFlash`. Input-Pin verbunden links, Output-Pin verbunden rechts.

> 📸 **Bild-Platzhalter:** `sideeffect-details-panel.png` — Details-Panel eines ApplyEffect-SideEffects.
> *Setup:* SideEffect-Eintrag aufklappen (Typ `UMayDlgSideEffect_ApplyEffect`). Sichtbar: `Description = "Ruf erhöhen"`, `EffectClass = GE_ReputationUp`, `EffectLevel = 2.0`, `bApplyToInstigator = true`.

> 📸 **Bild-Platzhalter:** `sideeffect-on-choice-pill.png` — Choice-Pill mit SideEffect-Pill auf einem PlayerChoice-Node.
> *Setup:* PlayerChoice-Node. Choice-Pill `"Angriff!"` aufgeklappt. Darin eine SideEffect-Pill `ApplyEffect: GE_Adrenaline`. Zeigt Verschachtelung: PlayerChoice → Choice → SideEffect.

## Mini-Beispiel

**SayLine mit mehreren Inline-Aktionen:**

```text
[SayLine: NPC | "Hier ist dein Dank, Held."]
  SideEffects:
    + ApplyEffect: GE_ReputationUp, Level=2, bApplyToInstigator=true
    + AddTag: Story.Guard.Grateful
    + TriggerCue: GameplayCue.UI.RewardFlash
```

Alle drei Aktionen laufen beim Betreten der SayLine, **bevor** der Text angezeigt wird.

**Choice mit SideEffect:**

```text
[PlayerChoice]
  Choice 0 "Ich kämpfe!"
    SideEffect: ApplyEffect GE_Adrenaline
    ──► [SayLine: "Dann los!"]
```

Der Effect wird beim Klick auf die Choice angewendet, bevor der Dialog zur SayLine springt.

> 📸 **Bild-Platzhalter:** `sideeffect-example-graph.png` — SayLine mit SideEffects und Übergang zu Exit.
> *Setup:* `Entry` → `SayLine "Hier ist dein Dank."` (drei SideEffect-Pills im Body) → `Exit: Completed`. Verbindungen sichtbar. Im Details-Panel der SayLine die drei SideEffects aufgeklappt.

## Action-Node vs. SideEffect

| Kriterium | Action-Node | SideEffect |
| --- | --- | --- |
| Haupt-Aktion dieses Schritts? | Ja | Nein |
| Soll im Graph als eigene Box sichtbar sein? | Ja | Nein |
| Debugger-Breakpoint darauf setzen? | Ja | Nein (nur am Eltern-Node) |
| Mehrere Aktionen am selben Moment? | Wird unübersichtlich | Ideal |

**Faustregel**: Bis zu zwei Inline-Aktionen → SideEffect-Pills. Drei oder mehr und/oder die Aktion ist der eigentliche Zweck des Schritts → eigener Action-Node.

## Häufige Fallstricke

- **SideEffects laufen vor `ExecuteNode`**: Wenn du in einem SideEffect auf den Zustand reagieren willst, der erst durch den Node geändert wird, kommt es zu einer falschen Reihenfolge. SideEffects laufen immer zuerst.
- **`bAddToInstigator` / `bApplyToInstigator` vergessen**: Ohne dieses Flag wirkt die Aktion auf das Target des Dialogs (z.B. NPC), nicht auf den Spieler. Setze es auf `true` für spielerbezogene Effekte.

## Erweitern

{% hint style="success" %}
**Eigene SideEffect-Logik in Blueprint:**

1. Rechtsklick im Content Browser → Blueprint Class → Parent: `UMayDialogueSideEffect`.
2. Event `ExecuteSideEffect` überschreiben — hier deine Spiellogik.
3. Optional: `ExecuteClientSideEffect` für rein kosmetische Aktionen (Sounds, Particles) überschreiben.
4. Blueprint kompilieren.
5. Im Details-Panel eines Nodes/Choices → "Add SideEffect" → deine neue Klasse erscheint in der Liste.

Details: [Extension → Custom SideEffects](../../extension/custom-side-effects.md).
{% endhint %}

```cpp
// C++-Override (Advanced):
virtual void ExecuteSideEffect_Implementation(const FMayDialogueContext& Context) override;
virtual void ExecuteClientSideEffect_Implementation(const FMayDialogueContext& Context) override;
```
