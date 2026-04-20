# GAS-Requirements

Drei vordefinierte Requirement-Klassen decken die häufigsten Dialog-GAS-Fragen ab. Alle liefern `EMayDialogueRequirementResult` (Passed / FailedButVisible / FailedAndHidden).

## UMayDlgRequirement_HasTag

Prüft, ob das Ziel einen bestimmten GameplayTag trägt.

### Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `RequiredTag` | `FGameplayTag` | Der zu prüfende Tag. |
| `bCheckOnInstigator` | bool | `true` → Spieler-ASC; `false` → NPC-ASC. |
| `bHideOnFail` | bool (Base) | FailedAndHidden vs. FailedButVisible bei Failure. |

### Runtime-Logik

1. Wenn Tag invalide → Passed.
2. Target-Actor auflösen (Instigator oder Target).
3. **Preferred**: ASC finden → `HasMatchingGameplayTag()`.
4. **Fallback**: Actor implementiert `IGameplayTagAssetInterface` → `GetOwnedGameplayTags()` → `HasTag()`.
5. Wenn gefunden: Passed. Sonst: FailureResult (`FailedAndHidden` oder `FailedButVisible`).

### Beispiel

```
Choice "Ich war dabei, als es passierte"
  Requirement: HasTag
    RequiredTag: Story.Witness.Murder
    bCheckOnInstigator: true
    bHideOnFail: true
```

Die Choice erscheint nur, wenn der Spieler den `Story.Witness.Murder`-Tag trägt.

## UMayDlgRequirement_CheckAttribute

Vergleicht ein GAS-Attribut gegen einen Schwellwert.

### Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `Attribute` | `FGameplayAttribute` | Welches Attribut (z.B. `UHealthAttributeSet::Health`). |
| `ComparisonValue` | float | Der Vergleichswert. |
| `ComparisonOp` | `EMayDlgComparisonOp` | `>` / `<` / `==` / `>=` / `<=` / `!=`. |
| `bCheckOnInstigator` | bool | Wer wird geprüft. |
| `FailureResult` | `EMayDialogueRequirementResult` | Default: `FailedButVisible`. |

### Runtime-Logik

1. Wenn Attribute invalide → Passed.
2. Target-ASC auflösen.
3. Prüfen, ob ASC ein AttributeSet mit diesem Attribut hat.
4. Aktuellen Numeric-Value lesen.
5. Vergleich: `CurrentValue ComparisonOp ComparisonValue`?
6. Passed oder `FailureResult`.

### Beispiel

```
Choice "Ich kann noch kämpfen"
  Requirement: CheckAttribute
    Attribute: Stamina (aus UVHSAttributeSet)
    ComparisonOp: >
    ComparisonValue: 30.0
    bCheckOnInstigator: true
    FailureResult: FailedButVisible
    UnavailableReason: "Du bist zu erschöpft."
```

Die Choice ist sichtbar, aber nicht wählbar, wenn Stamina ≤ 30. Tooltip erklärt warum.

## UMayDlgRequirement_HasAbility

Prüft, ob eine GameplayAbility-Klasse im ASC gegranted ist.

### Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `RequiredAbility` | `TSubclassOf<UGameplayAbility>` | Die Ability-Klasse. |
| `bCheckOnInstigator` | bool | Wer wird geprüft. |

### Runtime-Logik

1. Wenn Klasse leer → Passed.
2. Target-ASC auflösen.
3. `FindAbilitySpecFromClass(RequiredAbility)` auf dem ASC.
4. Gefunden → Passed. Sonst: FailureResult.

### Beispiel

```
Choice "Ich wirke einen Feuerball"
  Requirement: HasAbility
    RequiredAbility: GA_Fireball
    bCheckOnInstigator: true
    bHideOnFail: true
```

## Fail-Modi – Design-Muster

| Modus | Einsatz |
| --- | --- |
| **FailedAndHidden** | *„Du siehst die Option erst, wenn du sie hast."* Klassisch RPG. |
| **FailedButVisible** | *„Du siehst sie immer, aber erst mit genug Charisma wählbar."* Macht Progression sichtbar. |

Designer wählt pro Requirement. Beide sind legitim.

## Kombination mehrerer Requirements

Sub-Nodes werden als Array gehalten. `UMayDialogueRequirement::EvaluateAll()` merged:

* Alle Passed → Passed.
* Eine FailedAndHidden → FailedAndHidden (gewinnt).
* Sonst eine FailedButVisible → FailedButVisible.

Beispiel: Choice braucht beide Tags „Story.Friend" **und** Attribut Reputation ≥ 50. Falls einer fehlt, Choice ist FailedAndHidden (wenn HasTag-Requirement `bHideOnFail=true` hat).

## Eigene Requirements

Siehe [Eigene GAS-Nodes erstellen](extending.md).
