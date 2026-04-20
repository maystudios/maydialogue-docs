# SideEffect

Ein **SideEffect** ist eine Inline-Aktion, die ausgeführt wird, sobald der Eltern-Node betreten wird (oder im Fall eines Choice-SideEffects: wenn die Choice gewählt wird).

## Konzept

Nicht jede Aktion verdient eine eigene Box im Graph. Wenn eine SayLine nebenbei eine Variable setzen soll, wäre ein eigener SetVariable-Node Overkill. SideEffects leben als **Pills im Body** des Eltern-Nodes – sichtbar, aber nicht aufdringlich.

## Basis-Klasse

`UMayDialogueSideEffect` (Abstract, EditInlineNew, BlueprintType, Blueprintable):

| Property | Typ | Zweck |
| --- | --- | --- |
| `Description` | `FText` | Editor-Tooltip / Pill-Label. |

Virtuelle Methoden:

```cpp
virtual void ExecuteSideEffect(const FMayDialogueContext& Context);
virtual void ExecuteClientSideEffect(const FMayDialogueContext& Context);
virtual FText GetDisplayDescription() const;
```

## Standard-SideEffects (im MayDialogueGAS-Modul)

| Klasse | Aktion | Properties |
| --- | --- | --- |
| `UMayDlgSideEffect_AddTag` | LooseTag hinzufügen | `TagToAdd`, `bAddToInstigator` |
| `UMayDlgSideEffect_RemoveTag` | LooseTag entfernen | `TagToRemove`, `bRemoveFromInstigator` |
| `UMayDlgSideEffect_ApplyEffect` | GE anwenden | `EffectClass`, `EffectLevel`, `bApplyToInstigator` |
| `UMayDlgSideEffect_TriggerCue` | GameplayCue feuern | `CueTag`, `bTriggerOnInstigator` |

Details siehe [GAS → Aktionen](../../gas/actions.md).

## Wo einsetzen?

Als **Array** auf:

* Jedem `UMayDialogueNode_Base` – ausgeführt vor `ExecuteNode`.
* Jedem `UMayDialogueChoice` – ausgeführt bei `SelectChoice`.

## Typisches Pattern

SayLine mit mehreren Inline-Aktionen:

```
[SayLine: NPC "Hier ist dein Dank, Held."]
  SideEffects:
    + ApplyEffect: GE_ReputationUp, Level=2
    + AddTag: Story.Guard.Grateful
    + TriggerCue: GameplayCue.UI.RewardFlash
```

All das passiert beim Betreten der SayLine, **bevor** der Text angezeigt wird.

## Action-Node vs. SideEffect – Entscheidungshilfe

| Kriterium | Action-Node | SideEffect |
| --- | --- | --- |
| Haupt-Aktion des Schritts? | ✅ | ❌ |
| Eigene Box sinnvoll im Graph? | ✅ | ❌ |
| Breakpoint-Debugbar? | ✅ | ❌ (nur am Eltern-Node) |
| Raum im Body des Eltern-Nodes sparen? | ❌ | ✅ |
| Mehrere Inline-Aktionen am selben Moment? | ❌ (wird unübersichtlich) | ✅ |

Als Faustregel: **Die dritte + vierte + fünfte Inline-Aktion an derselben Stelle** sprengen den Graph – dann lohnen sich separate Action-Nodes. Bis zu zwei Inline-Aktionen sind als SideEffect-Pills perfekt.

## Eigene SideEffects

Per Blueprint:

1. Rechtsklick im Content Browser → Blueprint Class → `UMayDialogueSideEffect` als Parent.
2. `ExecuteSideEffect` überschreiben.
3. Blueprint kompilieren.

Siehe [Extension → Custom SideEffects](../../extension/custom-side-effects.md).
