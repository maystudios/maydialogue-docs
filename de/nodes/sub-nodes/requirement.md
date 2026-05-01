# Requirement

Ein Requirement ist ein Bedingungscheck, der drei Ergebnisse liefern kann: bestanden, nicht bestanden aber sichtbar, oder komplett ausgeblendet. Du setzt Requirements auf Nodes und Choices ein, um den Dialog-Fluss und die Verfügbarkeit von Optionen zu steuern.

## Wann setze ich ihn ein?

- Als Condition auf einem [Branch-Node](../core/branch.md), um automatisch zwischen Pfaden zu wählen.
- Auf einer [Choice](choice.md), um eine Option nur für Spieler mit dem richtigen Tag oder Attribut sichtbar zu machen.
- Auf einem beliebigen Node, um ihn zu überspringen, wenn der Spieler einen bestimmten Zustand nicht hat.
- Mehrere Requirements auf demselben Node/Choice werden kombiniert: das strengste Ergebnis gewinnt.

## Die drei Ergebnisse

| Ergebnis | Bedeutung |
| --- | --- |
| `Passed` | Bedingung erfüllt — Node/Choice wird normal ausgeführt/angezeigt. |
| `FailedButVisible` | Nicht erfüllt — Choice wird ausgegraut angezeigt (mit `UnavailableReason`-Tooltip). Node wird übersprungen oder Abort (je nach `FailBehavior`). |
| `FailedAndHidden` | Nicht erfüllt — Choice wird gar nicht angezeigt. Node wird übersprungen. |

## Properties (Basis-Klasse)

| Property | Typ | Standard | Bedeutung |
| --- | --- | --- | --- |
| `Description` | `FText` | leer | Editor-Tooltip / Pill-Label im Graph. |
| `FailResult` | `EMayDialogueRequirementFailResult` | `FailedButVisible` | Was passiert bei Nicht-Erfüllung: `FailedButVisible` oder `FailedAndHidden`. |

{% hint style="warning" %}
`bHideOnFail` ist veraltet — nutze stattdessen das `EMayDialogueRequirementResult`-Feld `FailResult = FailedAndHidden`.
{% endhint %}

## Standard-Requirements (GAS-Integration)

Diese Requirements sind direkt verfügbar — keine zusätzliche Einrichtung nötig. Sie greifen auf das Gameplay Ability System zu, wenn dein Projekt es verwendet.

| Klasse | Prüft | Wichtige Properties |
| --- | --- | --- |
| `UMayDlgRequirement_HasTag` | ASC trägt einen bestimmten Gameplay-Tag | `RequiredTag`, `bCheckOnInstigator` |
| `UMayDlgRequirement_CheckAttribute` | Attribut-Vergleich (>,<,==,>=,<=,!=) | `Attribute`, `ComparisonOp`, `ComparisonValue`, `bCheckOnInstigator` |
| `UMayDlgRequirement_HasAbility` | ASC hat eine bestimmte Ability-Klasse | `RequiredAbility`, `bCheckOnInstigator` |

Details: [GAS → Requirements](../../gas/requirements.md).

> 📸 **Bild-Platzhalter:** `requirement-node-pill.png` — Requirement als Pill auf einem Branch-Node.
> *Setup:* Branch-Node im Graph auswählen. Im Node-Body sichtbar: Condition-Pill `HasTag Story.Met.Guard`. Details-Panel rechts zeigt: `RequiredTag = Story.Met.Guard`, `FailResult = FailedButVisible`, `Description = "Hat der Spieler den Wächter bereits getroffen?"`.

> 📸 **Bild-Platzhalter:** `requirement-details-panel.png` — Details-Panel eines HasTag-Requirements.
> *Setup:* Requirement im Details-Panel des Branch-Nodes aufklappen. Sichtbar: `Description`, `FailResult = FailedAndHidden`, `RequiredTag = Story.HeardPassword`, `bCheckOnInstigator = true`.

> 📸 **Bild-Platzhalter:** `requirement-on-choice-pill.png` — Choice-Pill mit angehängtem Requirement (als Pill im Choice-Body).
> *Setup:* PlayerChoice-Node. Choice-Pill `"Passwort: 'Schattentür'."` aufgeklappt. Im Choice-Body sichtbar: Requirement-Pill `HasTag Story.HeardPassword (FailedAndHidden)`. Zeigt den Eltern-Node (PlayerChoice) mit angehängtem Sub-Node (Requirement) in der Pill.

## Mini-Beispiel

**Auf Branch (automatische Weiche):**

```text
[Branch]
  Condition: HasTag "Story.Met.Guard"
  ├─ True  ──► [SayLine: "Schön, dich wiederzusehen."]
  └─ False ──► [SayLine: "Halt! Wer bist du?"]
```

**Auf Choice (Availability):**

```text
[PlayerChoice]
  ├─ Choice 0 "Freund"                       (keine Requirements)
  ├─ Choice 1 "Passwort nennen"              (HasTag Story.HeardPassword, FailedAndHidden)
  └─ Choice 2 "Mit Ansehen überzeugen"       (CheckAttribute Reputation >= 50, FailedButVisible)
      UnavailableReason: "Dein Ansehen reicht nicht aus."
```

> 📸 **Bild-Platzhalter:** `requirement-example-graph.png` — PlayerChoice mit drei Choices und verschiedenen Requirements.
> *Setup:* PlayerChoice-Node, drei Choices aufgeklappt. Choice 0: keine Pill. Choice 1: Requirement-Pill `HasTag Story.HeardPassword`. Choice 2: Requirement-Pill `CheckAttribute Reputation >= 50`. `UnavailableReason` bei Choice 2 im Details-Panel sichtbar.

## Mehrere Requirements kombinieren

Requirements auf demselben Node/Choice werden mit `EvaluateAll` kombiniert:

- Alle `Passed` → `Passed`.
- Irgendeiner `FailedAndHidden` → `FailedAndHidden` (stärkstes Ergebnis gewinnt).
- Sonst irgendeiner `FailedButVisible` → `FailedButVisible`.

## Häufige Fallstricke

- **`FailResult` nicht gesetzt**: Standard ist `FailedButVisible` — die Choice/Node ist sichtbar, aber deaktiviert. Wenn du sie ganz ausblenden willst, stelle `FailResult = FailedAndHidden` ein.
- **bCheckOnInstigator vergessen**: Ohne dieses Flag prüft das Requirement am Target des Dialogs (z.B. NPC), nicht am Spieler. Setze `bCheckOnInstigator = true` für spielerbezogene Checks.

## Erweitern

{% hint style="success" %}
**Eigene Requirement-Logik in Blueprint:**

1. Rechtsklick im Content Browser → Blueprint Class → Parent: `UMayDialogueRequirement`.
2. Event `IsRequirementSatisfied` überschreiben — gibt `Passed`, `FailedButVisible` oder `FailedAndHidden` zurück.
3. Blueprint kompilieren.
4. Im Details-Panel eines Nodes/Choices → "Add Requirement" → deine neue Klasse erscheint in der Liste.

Details: [Extension → Custom Requirements](../../extension/custom-requirements.md).
{% endhint %}

```cpp
// C++-Override (Advanced):
virtual EMayDialogueRequirementResult IsRequirementSatisfied_Implementation(
    const FMayDialogueContext& Context) const override;
```
