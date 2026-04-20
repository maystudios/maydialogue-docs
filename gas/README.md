# GAS-Integration

Epic's **Gameplay Ability System** ist in MayDialogue erste-Klasse-Bürger. Requirements prüfen ASC-State, SideEffects mutieren ASC-State – alles ohne Custom-Code.

## Die drei Integrations-Säulen

1. **Requirements** prüfen ASC-State (Tags, Attribute, Abilities).
2. **SideEffects / Actions** mutieren ASC-State (Tags setzen/entfernen, Effects anwenden, Cues feuern).
3. **Helpers** resolven ASCs aus dem Dialog-Kontext, ohne Boilerplate.

## Modul-Lage

Alle GAS-Klassen leben im **`MayDialogueGAS`**-Sub-Modul (Runtime, LoadingPhase `Default`). Das Core-Modul hat keine Hard-Dependency auf GAS im Public-Header – GAS-Klassen werden per UE-Reflection automatisch in die Node-/Sub-Node-Picker eingeblendet, sobald das Modul geladen ist.

## Kapitel-Überblick

* [Requirements](requirements.md) – `HasTag`, `CheckAttribute`, `HasAbility`.
* [Aktionen](actions.md) – `AddTag`, `RemoveTag`, `ApplyEffect`, `TriggerCue`.
* [Helpers](helpers.md) – ASC-Resolution-Utilities.
* [Eigene GAS-Nodes erstellen](extending.md) – Blueprint-Erweiterung.

## Dialog-GAS-Kopplung: wo

| Stelle | GAS-Interaktion |
| --- | --- |
| Choice-Requirement | *„Nur wenn Spieler Tag X hat."* |
| Branch-Condition | *„Wenn Health < 20, gehe zu Verletzt-Variante."* |
| SayLine-Requirement | *„Diese Zeile nur wenn NPC Angst-Tag trägt."* |
| SideEffect an Node | *„Beim Betreten dieser SayLine: Reputation-Tag hinzufügen."* |
| Action-Node | *„Wende GE_HealMajor auf den Spieler an."* |

## Context-basierte Zielauflösung

Alle GAS-Klassen haben ein `bCheckOnInstigator` / `bAddToInstigator` / `bApplyToInstigator` / `bTriggerOnInstigator`-Flag. Der Flag bestimmt, ob der Spieler (Instigator) oder der NPC (Target) das Ziel ist:

| Flag | Ziel |
| --- | --- |
| `true` | Instigator (normalerweise Spieler) |
| `false` | Target (normalerweise NPC) |

## Beispiel: Passwort-Requirement

```
Choice "Ich kenne das Passwort"
  Requirement: HasTag
    - RequiredTag: Story.Secret.HeardPassword
    - bCheckOnInstigator: true
    - bHideOnFail: true  (Choice wird komplett ausgeblendet)
```

Wenn der Spieler den Tag auf seinem ASC hat (z.B. weil er vorher mit einem NPC gesprochen hat, der ihm das Passwort gegeben hat), erscheint die Choice. Sonst: unsichtbar.

## Beispiel: Heal-Reward

```
Node ApplyEffect (Action-Node)
  EffectClass: GE_HealMajor
  EffectLevel: 1.0
  bApplyToInstigator: true
```

Wendet `GE_HealMajor` auf den Spieler an (Instigator-ASC ist gleichzeitig Source und Target).

## Ohne GAS im Projekt

Wenn dein Projekt **kein GAS** nutzt: die GAS-Klassen sind trotzdem ladbar, werden aber silently no-op, wenn kein ASC gefunden wird. Du kannst sie also ignorieren und stattdessen mit Dialog-/Participant-Variablen arbeiten.

## Mit GAS, aber ohne MayDialogueGAS-Modul

Theoretisch kann das `MayDialogueGAS`-Modul abgeschaltet werden. Dann fehlen die GAS-Requirements/SideEffects in den Node-Pickern. Für GAS-Funktionen müsstest du eigene Requirement-/SideEffect-Subklassen bauen.

In der Praxis: Modul eingeschaltet lassen.
