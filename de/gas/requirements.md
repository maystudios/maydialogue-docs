---
description: HasTag, CheckAttribute, HasAbility — drei vordefinierte GAS-Requirements für Dialog-Bedingungen.
---

# GAS-Requirements

Requirements entscheiden, ob ein Choice, Branch oder SayLine-Node aktiv ist. Diese drei Klassen decken die häufigsten GAS-Prüfungen ab — ohne eine Zeile Code.

Alle Requirements liefern einen von drei Zuständen:

| Ergebnis | Bedeutung |
| --- | --- |
| `Passed` | Bedingung erfüllt — Node ist aktiv. |
| `FailedButVisible` | Nicht erfüllt, aber sichtbar (z.B. ausgegraut mit Tooltip). |
| `FailedAndHidden` | Nicht erfüllt und komplett unsichtbar. |

---

## Has Gameplay Tag

**Klasse:** `UMayDlgRequirement_HasTag`

Prüft, ob ein Actor einen bestimmten GameplayTag trägt. Zuerst über den AbilitySystemComponent, als Fallback über `IGameplayTagAssetInterface`.

### Wofür

Klassischer Use-Case: eine Choice erscheint nur, wenn der Spieler einen Story-Tag erworben hat — z.B. weil er ein Codex-Fragment gefunden oder zuvor mit einem Informanten gesprochen hat.

### Properties

| Property | Typ | Beschreibung |
| --- | --- | --- |
| `RequiredTag` | `FGameplayTag` | Der Tag, der geprüft wird. |
| `bCheckOnInstigator` | bool | `true` = Spieler-ASC; `false` = NPC-ASC. |
| `bHideOnFail` | bool | `true` = FailedAndHidden; `false` = FailedButVisible. |

> 📸 **Bild-Platzhalter:** `req-hastag-details.png` — Details-Panel des HasTag-Requirements.
> *Setup:* Auf einer Choice den HasTag-Sub-Node auswählen. Details-Panel rechts zeigt: `RequiredTag = Story.Witness.Murder`, `bCheckOnInstigator = true` (Häkchen gesetzt), `bHideOnFail = true` (Häkchen gesetzt). Felder klar beschriftet, kein anderer Sub-Node sichtbar.

### Beispiel: Zeugen-Choice

```text
Choice "Ich war dabei, als es passierte"
  Requirement: Has Gameplay Tag
    RequiredTag:        Story.Witness.Murder
    bCheckOnInstigator: true
    bHideOnFail:        true
```

Die Choice erscheint nur, wenn der Spieler-ASC den Tag `Story.Witness.Murder` trägt.

### Fail-Modi im Vergleich

| Modus | Wann nutzen |
| --- | --- |
| `bHideOnFail = true` | Klassisches RPG: Option existiert erst, wenn Voraussetzung erfüllt. |
| `bHideOnFail = false` | Skill-System: Option ist immer sichtbar, macht Progression greifbar. |

---

## Check GAS Attribute

**Klasse:** `UMayDlgRequirement_CheckAttribute`

Vergleicht ein GAS-Attribut gegen einen Schwellwert. Unterstützt alle gängigen Vergleichsoperatoren und kann BaseValue oder CurrentValue lesen.

### Wofür

Checks wie "Spieler hat genug Stamina zum Kämpfen" oder "NPC hat weniger als 30% Health". Besonders nützlich für Choices, die sichtbar, aber deaktiviert sein sollen, um Progression anzuzeigen.

### Properties

| Property | Typ | Beschreibung |
| --- | --- | --- |
| `Attribute` | `FGameplayAttribute` | Das zu prüfende Attribut (z.B. `UVHSAttributeSet::GetStaminaAttribute()`). |
| `ComparisonValue` | float | Schwellwert. |
| `ComparisonOp` | `EMayDlgComparisonOp` | `>` / `<` / `==` / `>=` / `<=` / `!=` |
| `bCheckOnInstigator` | bool | `true` = Spieler; `false` = NPC. |
| `bUseBaseValue` | bool | `true` = BaseValue (vor Modifier); `false` = CurrentValue (nach Modifier). |
| `Tolerance` | float | Toleranz für `==` / `!=`-Vergleiche (verhindert Floating-Point-Fallen). |
| `FailResult` | `EMayDialogueRequirementResult` | Was bei Failure zurückgegeben wird. |

> 📸 **Bild-Platzhalter:** `req-checkattribute-details.png` — Details-Panel des CheckAttribute-Requirements mit gefüllten Werten.
> *Setup:* CheckAttribute-Sub-Node auf einer Choice auswählen. Details-Panel zeigt: `Attribute = Stamina (UVHSAttributeSet)`, `ComparisonOp = GreaterOrEqual (>=)`, `ComparisonValue = 30.0`, `bCheckOnInstigator = true`, `bUseBaseValue = false`, `Tolerance = 0.0001`, `FailResult = FailedButVisible`.

### Beispiel: Stamina-Check

```text
Choice "Ich kann noch kämpfen"
  Requirement: Check GAS Attribute
    Attribute:          Stamina (aus deinem AttributeSet)
    ComparisonOp:       >= (GreaterOrEqual)
    ComparisonValue:    30.0
    bCheckOnInstigator: true
    FailResult:         FailedButVisible
```

Die Choice ist sichtbar, aber nicht wählbar, wenn Stamina ≤ 30. Ein optionaler Tooltip erklärt dem Spieler, warum.

{% hint style="info" %}
**BaseValue vs. CurrentValue:** Nutze `bUseBaseValue = true`, wenn du auf den "permanenten" Wert prüfen willst — unabhängig davon, ob gerade ein Buff oder Debuff aktiv ist. Für "wie es sich gerade anfühlt" nutze `bUseBaseValue = false` (Standard).
{% endhint %}

---

## Has Gameplay Ability

**Klasse:** `UMayDlgRequirement_HasAbility`

Prüft, ob eine GameplayAbility auf dem ASC gegranted ist. Kann per Klasse oder per Tag-Container matchen.

### Wofür

Zeige Choices nur an, wenn der Spieler eine bestimmte Ability hat ("Ich wirke einen Feuerball" → nur wenn `GA_Fireball` gegranted ist). Oder prüfe, ob eine Ability gerade aktiv läuft.

### Properties

| Property | Typ | Beschreibung |
| --- | --- | --- |
| `RequiredAbility` | `TSubclassOf<UGameplayAbility>` | Klassen-basierter Match (Fallback wenn Tag-Container leer). |
| `AbilityTagsAny` | `FGameplayTagContainer` | Passt, wenn eine Ability **mindestens einen** dieser Tags hat. |
| `AbilityTagsAll` | `FGameplayTagContainer` | Passt, wenn eine Ability **alle** dieser Tags hat. |
| `bRequireActive` | bool | `true` = nur Abilities zählen, die gerade aktiv laufen (`IsActive()`). |
| `bCheckOnInstigator` | bool | `true` = Spieler; `false` = NPC. |
| `bHideOnFail` | bool | Sichtbarkeitsmodus bei Failure. |

**Match-Priorität:** `AbilityTagsAny` → `AbilityTagsAll` → `RequiredAbility`. Die Klassen-Prüfung ist nur aktiv, wenn beide Tag-Container leer sind.

> 📸 **Bild-Platzhalter:** `req-hasability-details.png` — Details-Panel des HasAbility-Requirements mit Tag-Container-Eintrag.
> *Setup:* HasAbility-Sub-Node auf einer Choice auswählen. Details-Panel zeigt: `RequiredAbility = leer`, `AbilityTagsAny = (Ability.Magic.Fire)`, `AbilityTagsAll = leer`, `bRequireActive = false`, `bCheckOnInstigator = true`, `bHideOnFail = true`.

### Beispiel: Magie-Choice

```text
Choice "Ich wirke einen Feuerball"
  Requirement: Has Gameplay Ability
    AbilityTagsAny:     Ability.Magic.Fire
    bCheckOnInstigator: true
    bHideOnFail:        true
```

Erscheint nur, wenn der Spieler-ASC eine Ability mit dem Tag `Ability.Magic.Fire` hat.

---

## Mehrere Requirements kombinieren

Auf einer Choice, einem Branch oder einer SayLine kannst du beliebig viele Requirements stapeln. Das System merged sie automatisch:

* Alle Passed → Passed.
* Mindestens einer `FailedAndHidden` → FailedAndHidden (dominiert).
* Sonst mindestens einer `FailedButVisible` → FailedButVisible.

```text
Choice "Ich bin der Auserwählte"
  Requirements:
    1. Has Gameplay Tag:     Story.Chosen.Marked        bHideOnFail: true
    2. Check GAS Attribute:  Reputation >= 80           FailResult: FailedAndHidden
```

Fehlt einer der beiden: Choice ist unsichtbar. Sind beide erfüllt: Choice erscheint.

{% hint style="info" %}
**Eigene Requirements bauen?** Lege eine Blueprint-Klasse mit Parent `UMayDialogueRequirement` an und überschreibe `Is Requirement Satisfied`. Mehr dazu in [Eigene GAS-Nodes erstellen](extending.md).
{% endhint %}
