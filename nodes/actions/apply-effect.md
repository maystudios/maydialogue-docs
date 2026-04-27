---
description: Einen UGameplayEffect auf Spieler oder Ziel-NPC anwenden — direkt aus dem Dialog.
---

# Apply Effect

Wendet einen `UGameplayEffect` auf den Instigator (Spieler) oder einen Ziel-Participant an. Nützlich überall dort, wo ein Dialog eine echte Gameplay-Konsequenz auslösen soll.

## Wann nutzen

- **NPC heilt Spieler** — Heilerin spricht Gebet, `GE_Heal` wird auf den Instigator angewendet.
- **Vergiftung als Strafe** — Spieler trinkt verdächtigen Trank auf Anraten eines NPC, `GE_Poison` trifft den Spieler.
- **Ruf-System** — Spieler gibt dem Händler Geld, `GE_ReputationUp` erhöht Ruf-Attribut am NPC (`bApplyToInstigator=false`).
- **Buff nach Ritual** — Langer Dialog-Abschluss verleiht Spieler temporären Stärke-Buff.

---

> 📸 **Bild-Platzhalter:** `apply-effect-node.png` — Node "Apply Effect" im MayDialogue-Graphen.
> *Setup:* Node allein sichtbar, Title-Bar "Apply Effect" (Kategorie-Farbe: lila/GAS). Subtitle zeigt: `EffectClass = GE_Heal`, `bApplyToInstigator = true`. Input- und Output-Pin sichtbar.

---

## Properties

| Property | Typ | Beschreibung |
|---|---|---|
| `EffectClass` | `TSubclassOf<UGameplayEffect>` | Der GameplayEffect der angewendet wird. |
| `EffectLevel` | `float` | Effect-Level für Magnitude-Scaling. Minimum: 0. Default: `1.0`. |
| `bApplyToInstigator` | `bool` | `true` = Spieler (Self-Buff/-Debuff). `false` = Ziel-NPC. |

---

> 📸 **Bild-Platzhalter:** `apply-effect-details.png` — Details-Panel mit gefüllten Werten.
> *Setup:* Node auswählen. Im Details-Panel sichtbar: `EffectClass = GE_ReputationUp`, `EffectLevel = 1.0`, `bApplyToInstigator = false`.

---

## Action-Node oder SideEffect-Sub-Node?

Wenn das Anwenden des Effects die **spielmechanische Hauptaussage** dieses Schrittes ist (z.B. "jetzt wird der Spieler geheilt"), nimm den Action-Node. Wenn der Effect nur im Hintergrund als Nebenwirkung einer SayLine läuft (z.B. leichte Stamina-Regeneration beim Zuhören), hänge ihn als SideEffect-Pill an die SayLine.

---

## Beispiel: Heilerin heilt Spieler

```text
[SayLine: Heilerin "Lass mich deine Wunden sehen."]
  │
  ▼
[ApplyEffect: EffectClass=GE_HealLarge, Level=1.0, bApplyToInstigator=true]
  │
  ▼
[SayLine: Heilerin "Jetzt geht es dir besser."]
```

> 📸 **Bild-Platzhalter:** `apply-effect-example-graph.png` — Graphausschnitt des obigen Beispiels.
> *Setup:* Drei Nodes: SayLine (Healerin) → ApplyEffect (GE_HealLarge sichtbar im Subtitle) → SayLine (Healerin). Pins verbunden.

---

## Fallstricke

{% hint style="warning" %}
Der Node ist ein **stiller No-Op** wenn `EffectClass` leer ist oder kein `AbilitySystemComponent` gefunden wird. Du siehst eine Log-Warning — aber keinen Fehler im Editor. Prüfe immer, dass Instigator und Target einen ASC haben.
{% endhint %}

{% hint style="info" %}
**Instigator ist immer der Source des Effects** — auch bei `bApplyToInstigator=false`. Das ist relevant wenn dein `UGameplayEffect` eine `MMC_*`-Berechnung verwendet, die Source-Attribute ausliest (z.B. `Source.Strength`).
{% endhint %}

- Der Node erfordert das **MayDialogueGAS-Modul** — ohne dieses Modul ist der Node nicht verfügbar.
- GE-Level `0.0` ist möglich, führt aber bei vielen Standard-Magnitude-Formeln zu einem Nullwert-Effect.
