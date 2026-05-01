---
description: Wie MayDialogue und das Gameplay Ability System zusammenspielen.
---

# GAS-Integration

MayDialogue kennt den Zustand deines Gameplay Ability Systems — und kann ihn verändern. Requirements lesen, SideEffects schreiben, alles ohne eigenen C++-Code.

## Warum GAS im Dialog?

Dialog-Entscheidungen hängen oft von Spielzustand ab: Hat der Spieler die richtige Ability? Ist seine Ausdauer hoch genug? Hat er gerade einen Quest-Tag? Statt diese Logik in Blueprint-Event-Graphen zu verdrahten, trägst du die Bedingung direkt in den Dialog-Knoten — als Requirement auf einer Choice oder einem Branch.

Und wenn der Dialog einen GAS-Effekt auslösen soll (Heilung, Tag vergeben, Cue abspielen), läuft das als SideEffect oder Action-Node — kein Glue-Code nötig.

## Die drei Integrations-Säulen

| Säule | Was sie tut |
| --- | --- |
| **Requirements** | Prüfen den ASC-State: Tag vorhanden? Attribut groß genug? Ability gegrant? |
| **SideEffects / Actions** | Mutieren den ASC-State: Tag hinzufügen/entfernen, GameplayEffect anwenden, GameplayCue feuern. |
| **Helpers** | Lösen ASCs aus dem Dialog-Kontext auf — kein Boilerplate, keine manuellen Casts. |

## Modul: MayDialogueGAS

Alle GAS-Klassen leben im separaten **`MayDialogueGAS`**-Modul. Das Core-Modul hat keine harte GAS-Abhängigkeit. Sobald `MayDialogueGAS` lädt, erscheinen die GAS-Klassen automatisch in den Node- und Sub-Node-Pickern des Editors.

{% hint style="info" %}
**Kein GAS im Projekt?** Die GAS-Klassen sind trotzdem ladbar, werden aber ignoriert, wenn kein ASC gefunden wird. Du kannst das Modul in Ruhe geladen lassen und arbeitest einfach mit Dialog-Variablen statt GAS-Nodes.
{% endhint %}

## Wo GAS in den Dialog eingreift

| Einsatzort | Beispiel |
| --- | --- |
| Choice-Requirement | *Nur wenn der Spieler Tag `Story.HeardPassword` trägt* |
| Branch-Condition | *Wenn Health < 20 → gehe zu Verletzt-Variante* |
| SayLine-Requirement | *Diese Zeile nur wenn NPC den Tag `NPC.IsAngry` trägt* |
| SideEffect an Node | *Beim Betreten dieser SayLine: Reputation-Tag setzen* |
| Action-Node | *GE_HealMajor auf den Spieler anwenden* |

## Instigator vs. Target

Alle GAS-Klassen haben ein Flag, das bestimmt, auf welchem Actor geprüft bzw. gewirkt wird:

| Flagwert | Ziel |
| --- | --- |
| `true` | Instigator — normalerweise der Spieler |
| `false` | Target — normalerweise der NPC |

Das Flag heißt je nach Klasse `bCheckOnInstigator`, `bAddToInstigator`, `bApplyToInstigator` oder `bTriggerOnInstigator` — semantisch immer dasselbe Konzept.

> 📸 **Bild-Platzhalter:** `gas-overview-table.png` — Übersichtstabelle im MayDialogue-Graph-Editor.
> *Setup:* Ein Dialog-Asset mit einer Choice öffnen. Die Choice hat ein `HasTag`-Requirement (Instigator = true) und ein `AddTag`-SideEffect. Im rechten Details-Panel beide Sub-Nodes sichtbar. Annotationen zeigen "Requirement prüft Instigator-ASC" und "SideEffect schreibt Instigator-ASC".

## Schnell-Beispiel: Passwort-Choice

```text
Choice "Ich kenne das Passwort"
  Requirement: HasTag
    RequiredTag: Story.Secret.HeardPassword
    bCheckOnInstigator: true
    bHideOnFail: true          ← Choice komplett ausblenden, wenn Tag fehlt
```

Hat der Spieler den Tag (z.B. weil er vorher mit einem Informanten gesprochen hat), erscheint die Choice. Sonst ist sie unsichtbar.

## Schnell-Beispiel: Heal-Reward als Action-Node

```text
[SayLine "Danke, du hast uns gerettet."]
  │
  ▼
[ApplyEffect]
  EffectClass: GE_HealMajor
  EffectLevel: 1.0
  bApplyToInstigator: true    ← Spieler wird geheilt
  │
  ▼
[Exit Completed]
```

> 📸 **Bild-Platzhalter:** `gas-heal-reward-graph.png` — Fertig verdrahteter Graph mit ApplyEffect-Node.
> *Setup:* Asset `DA_Reward_Heal` öffnen. Sichtbar: `SayLine`-Node (dunkelrot, Text "Danke…") → `ApplyEffect`-Node (blau, EffectClass = GE_HealMajor, EffectLevel = 1.0, bApplyToInstigator = true) → `Exit`-Node (rot). Verbindungen horizontal von links nach rechts.

## Kapitel-Überblick

* [Requirements](requirements.md) — `HasTag`, `CheckAttribute`, `HasAbility`
* [Aktionen](actions.md) — `AddTag`, `RemoveTag`, `ApplyEffect`, `TriggerCue`
* [Helpers](helpers.md) — ASC-Auflösung, Context-Utilities
* [Eigene GAS-Nodes erstellen](extending.md) — Blueprint-Erweiterung Schritt für Schritt

> 📸 **Bild-Platzhalter:** `gas-module-picker.png` — Node-Picker im Dialog-Editor mit GAS-Kategorie ausgeklappt.
> *Setup:* Im MayDialogue-Editor Rechtsklick in den Graphen. Kontext-Menü zeigt Kategorie `GAS` mit den Einträgen `Has Gameplay Tag`, `Check GAS Attribute`, `Has Gameplay Ability`, `Add Gameplay Tag`, `Remove Gameplay Tag`, `Apply Gameplay Effect`, `Trigger Gameplay Cue`. Kategorie ausgeklappt, alle Einträge sichtbar.
