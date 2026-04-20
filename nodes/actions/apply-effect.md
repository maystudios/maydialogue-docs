# Apply Effect

Wendet einen **GAS-`UGameplayEffect`** an. Der Instigator ist die Quelle, das Ziel ist der Instigator selbst oder der Target.

## Runtime-Verhalten

`ExecuteNode` (im MayDialogueGAS-Modul):

1. Holt den **Source-ASC** vom Instigator.
2. Erzeugt EffectContext mit `MakeEffectContext()`.
3. Erzeugt Spec mit `MakeOutgoingSpec(EffectClass, EffectLevel, Context)`.
4. Ermittelt Target-ASC je nach `bApplyToInstigator`.
5. `ApplyGameplayEffectSpecToTarget(Spec, TargetASC)`.
6. Advance.

## Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `EffectClass` | `TSubclassOf<UGameplayEffect>` | GE-Klasse. |
| `EffectLevel` | `float` | Effect-Level (z.B. für magnitude scaling). Clamped ≥ 0. Default: 1.0. |
| `bApplyToInstigator` | `bool` | `true` → Self-Buff; `false` → Target. |

## Typisches Pattern

Reputation erhöhen:

```
[PlayerChoice: "Hier, als Dank." (Geld übergeben)]
  │
  ▼
[ApplyEffect: GE_ReputationUp, Level=1.0, bApplyToInstigator=false]
  │
  ▼
[SayLine: NPC "Sehr freundlich von dir."]
```

## Anmerkungen

* **Instigator ist immer der Source** des Effects – auch bei Self-Buffs. Das ist wichtig für GE-Modifier-Magnitude-Calculations, die Source-Werte konsumieren.
* Funktioniert nur, wenn sowohl Instigator als auch Target einen `UAbilitySystemComponent` haben (direkt oder via `IAbilitySystemInterface`).
* Silently No-Op, wenn Effect-Klasse leer oder kein ASC auffindbar (Log-Warning).
