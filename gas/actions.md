# GAS-Aktionen

Vier vordefinierte GAS-SideEffect-Klassen. Jede existiert **in zwei Formen**:

* Als [**Action-Node**](../nodes/actions/README.md) (eigene Box im Graph) – für Hauptschritte.
* Als [**SideEffect-Sub-Node**](../nodes/sub-nodes/side-effect.md) (Pill im Body) – für Nebenschritte.

Die Logik ist identisch. Der Unterschied ist rein visuell.

## UMayDlgSideEffect_AddTag

Fügt einen LooseGameplayTag zum ASC hinzu.

| Property | Typ |
| --- | --- |
| `TagToAdd` | `FGameplayTag` |
| `bAddToInstigator` | `bool` |

**Logik**:

1. Tag validieren.
2. ASC via `GetASCFromContext(Context, bAddToInstigator)`.
3. `ASC->AddLooseGameplayTag(TagToAdd)`.

## UMayDlgSideEffect_RemoveTag

Entfernt einen LooseGameplayTag.

| Property | Typ |
| --- | --- |
| `TagToRemove` | `FGameplayTag` |
| `bRemoveFromInstigator` | `bool` |

**Logik**:

1. Tag validieren.
2. ASC auflösen.
3. `ASC->RemoveLooseGameplayTag(TagToRemove)`.

## UMayDlgSideEffect_ApplyEffect

Wendet einen `UGameplayEffect` an.

| Property | Typ |
| --- | --- |
| `EffectClass` | `TSubclassOf<UGameplayEffect>` |
| `EffectLevel` | `float` (≥ 0, default 1.0) |
| `bApplyToInstigator` | `bool` |

**Logik**:

1. EffectClass validieren.
2. **Source-ASC = Instigator** (immer).
3. Target-ASC je nach `bApplyToInstigator`.
4. `MakeEffectContext()`, `MakeOutgoingSpec(EffectClass, Level, Context)`.
5. `ApplyGameplayEffectSpecToTarget(Spec, TargetASC)`.

## UMayDlgSideEffect_TriggerCue

Feuert einen GameplayCue one-shot.

| Property | Typ |
| --- | --- |
| `CueTag` | `FGameplayTag` (unter `GameplayCue.*`) |
| `bTriggerOnInstigator` | `bool` |

**Logik**:

1. CueTag validieren.
2. ASC auflösen.
3. `FGameplayCueParameters` mit Instigator + EffectCauser.
4. `ASC->ExecuteGameplayCue(CueTag, Params)`.

## Aktions-Node vs. SideEffect

Jede dieser Klassen ist sowohl Action-Node als auch SideEffect-Sub-Node verfügbar.

### Als Action-Node (prominent)

```
[SayLine]
  │
  ▼
[ApplyEffect-Action]
  EffectClass = GE_BigBuff
  │
  ▼
[SayLine]
```

### Als SideEffect-Sub-Node (kompakt)

```
[SayLine]
  SideEffects:
    + ApplyEffect: GE_BigBuff
    + AddTag: Story.Buffed
    + TriggerCue: GameplayCue.UI.BuffFlash
```

## Design-Regel: Wann welche Form?

* **Haupt-Aktion** dieses Graph-Schritts → Action-Node.
* **Nebenbei** beim Betreten eines Nodes → SideEffect-Sub-Node.

Beide dürfen im selben Graph koexistieren.
