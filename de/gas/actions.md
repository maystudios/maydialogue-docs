---
description: AddTag, RemoveTag, ApplyEffect, TriggerCue — als Action-Node oder SideEffect.
---

# GAS-Aktionen

Vier vordefinierte Aktionen decken die häufigsten GAS-Mutationen im Dialog ab. Jede existiert in zwei Formen — die Logik ist identisch, nur der visuelle Platz im Graphen unterscheidet sich.

| Form | Wann nutzen |
| --- | --- |
| **Action-Node** (eigene Box im Graphen) | Die Aktion ist der Hauptschritt dieses Graphbereichs |
| **SideEffect-Sub-Node** (Pill an einem anderen Node) | Die Aktion passiert nebenbei beim Betreten eines Nodes |

---

## Add Gameplay Tag

**Klasse:** `UMayDlgSideEffect_AddTag`

Fügt einen Loose-GameplayTag zum ASC hinzu. Der Tag bleibt, bis er explizit wieder entfernt wird — er ist nicht an einen GameplayEffect gebunden.

### Properties

| Property | Typ | Beschreibung |
| --- | --- | --- |
| `TagToAdd` | `FGameplayTag` | Der hinzuzufügende Tag. |
| `bAddToInstigator` | bool | `true` = Spieler-ASC; `false` = NPC-ASC. |

> 📸 **Bild-Platzhalter:** `action-addtag-details.png` — Details-Panel des AddTag-Sub-Nodes.
> *Setup:* SayLine-Node öffnen, SideEffect-Array aufklappen, `AddTag`-Sub-Node auswählen. Details-Panel zeigt: `TagToAdd = Story.Accepted.QuestHelp`, `bAddToInstigator = true`.

### Beispiel: Story-Flag setzen

```text
SayLine "Natürlich helfe ich euch."
  SideEffects:
    + Add Gameplay Tag
        TagToAdd:       Story.Accepted.QuestHelp
        bAddToInstigator: true
```

Ab diesem Moment trägt der Spieler-ASC den Tag — andere Dialoge können darauf prüfen.

---

## Remove Gameplay Tag

**Klasse:** `UMayDlgSideEffect_RemoveTag`

Entfernt einen Loose-GameplayTag vom ASC. Funktioniert nur für Tags, die per `AddLooseGameplayTag` gesetzt wurden, nicht für Tags aus aktiven GameplayEffects.

### Properties

| Property | Typ | Beschreibung |
| --- | --- | --- |
| `TagToRemove` | `FGameplayTag` | Der zu entfernende Tag. |
| `bRemoveFromInstigator` | bool | `true` = Spieler-ASC; `false` = NPC-ASC. |

### Beispiel: Tarnung aufheben

```text
SayLine "Ich erkenne dich — weg mit der Verkleidung!"
  SideEffects:
    + Remove Gameplay Tag
        TagToRemove:         Story.Disguised
        bRemoveFromInstigator: true
```

---

## Apply Gameplay Effect

**Klasse:** `UMayDlgSideEffect_ApplyEffect`

Wendet einen `UGameplayEffect` auf einen ASC an. Source-ASC ist immer der Instigator. Target-ASC wird per Flag bestimmt.

### Properties

| Property | Typ | Beschreibung |
| --- | --- | --- |
| `EffectClass` | `TSubclassOf<UGameplayEffect>` | Die anzuwendende Effect-Klasse. |
| `EffectLevel` | float | Level des Effects (≥ 0, Standard 1.0). |
| `bApplyToInstigator` | bool | `true` = auf Spieler anwenden; `false` = auf NPC anwenden. |

> 📸 **Bild-Platzhalter:** `action-applyeffect-details.png` — Details-Panel des ApplyEffect-Nodes.
> *Setup:* ApplyEffect-Action-Node im Graphen auswählen. Details-Panel zeigt: `EffectClass = GE_HealMajor`, `EffectLevel = 1.0`, `bApplyToInstigator = true`. EffectClass-Picker zeigt den BP-Namen des Effects.

### Als Action-Node (Hauptschritt)

```text
[SayLine "Für deine Treue — nimm diesen Segen."]
  │
  ▼
[Apply Gameplay Effect]
  EffectClass: GE_BlessingBuff
  EffectLevel: 1.0
  bApplyToInstigator: true
  │
  ▼
[Exit Completed]
```

### Als SideEffect (Nebenschritt)

```text
SayLine "Ein kleines Dankeschön..."
  SideEffects:
    + Apply Gameplay Effect
        EffectClass:       GE_SmallHeal
        EffectLevel:       1.0
        bApplyToInstigator: true
```

{% hint style="warning" %}
**Auf den NPC anwenden:** Setze `bApplyToInstigator = false`, um den Effect auf den NPC zu spielen. Source-ASC bleibt immer der Spieler — das ist die Standard-GAS-Konvention für "Spieler heilt NPC".
{% endhint %}

---

## Trigger Gameplay Cue

**Klasse:** `UMayDlgSideEffect_TriggerCue`

Feuert einen GameplayCue auf einem ASC. Unterstützt drei Modi: einmaliges Execute, persistentes Add und Remove.

### Properties

| Property | Typ | Beschreibung |
| --- | --- | --- |
| `CueTag` | `FGameplayTag` | Muss unter der `GameplayCue.*`-Hierarchie liegen. |
| `bTriggerOnInstigator` | bool | `true` = Spieler-ASC; `false` = NPC-ASC. |
| `Mode` | `EMayDlgCueMode` | `Execute` (One-Shot) / `Add` (persistent) / `Remove` |

| Modus | Wann nutzen |
| --- | --- |
| `Execute` | Kurzer visueller oder akustischer Effekt, z.B. Aufleuchten, Klingeln. |
| `Add` | Dauerhafter visueller Zustand, z.B. Heiligenschein, der bleibt bis `Remove`. |
| `Remove` | Vorher per `Add` gestarteten Cue wieder abschalten. |

> 📸 **Bild-Platzhalter:** `action-triggercue-details.png` — Details-Panel des TriggerCue-Sub-Nodes.
> *Setup:* TriggerCue-SideEffect auf einem SayLine-Node auswählen. Details-Panel zeigt: `CueTag = GameplayCue.UI.BuffFlash`, `bTriggerOnInstigator = true`, `Mode = Execute (One-Shot)`.

### Beispiel: Visuelles Feedback beim Buff

```text
SayLine "Spüre die Kraft der alten Magie!"
  SideEffects:
    + Apply Gameplay Effect
        EffectClass:       GE_AncientStrengthBuff
        bApplyToInstigator: true
    + Trigger Gameplay Cue
        CueTag:            GameplayCue.UI.BuffFlash
        bTriggerOnInstigator: true
        Mode:              Execute
```

---

## Wann Action-Node, wann SideEffect?

```text
Action-Node:
[SayLine "Hier ist deine Bezahlung."]
  │
  ▼
[Apply Gameplay Effect]    ← Hauptschritt, verdient eigene Box
  EffectClass: GE_GoldReward
  │
  ▼
[Exit]

SideEffect:
[SayLine "Dankeschön!"]    ← Hauptschritt ist die Zeile
  SideEffects:
    + Add Tag: Story.NPC.ThankYouSaid     ← passiert nebenbei
    + Trigger Cue: GameplayCue.Speech.Thanks
```

**Faustregel:** Wenn der GAS-Effekt der Grund ist, warum dieser Graph-Abschnitt existiert → Action-Node. Wenn er begleitend beim Betreten eines anderen Nodes passiert → SideEffect.

> 📸 **Bild-Platzhalter:** `action-vs-sideeffect-graph.png` — Zwei Graph-Ausschnitte nebeneinander: links Action-Node als eigene Box, rechts dieselbe Logik als SideEffect-Pill an einer SayLine.
> *Setup:* Zwei Beispiel-Graphen im selben Asset nebeneinander legen. Linke Seite: `SayLine → ApplyEffect-Node (blau, groß) → Exit`. Rechte Seite: `SayLine` mit aufgeklapptem SideEffects-Array, darin `ApplyEffect`-Pill. Beide Seiten beschriften: "Als Action-Node" und "Als SideEffect".

{% hint style="info" %}
**Eigene GAS-Aktionen bauen?** Erstelle eine Blueprint-Subklasse von `UMayDialogueSideEffect` und überschreibe `Execute Side Effect`. Mehr dazu in [Eigene GAS-Nodes erstellen](extending.md).
{% endhint %}
