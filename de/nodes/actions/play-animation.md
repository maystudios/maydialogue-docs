---
description: Eine AnimMontage auf einem Dialogue-Participant abspielen — mit oder ohne Warten.
---

# Play Animation

Spielt eine `UAnimMontage` auf einem Participant ab. Du kannst die Wiedergabe-Geschwindigkeit und eine Start-Section wählen. Optional wartet der Dialog auf das Ende der Montage bevor er weitermacht.

## Wann nutzen

- **NPC-Geste beim Zustimmen** — Kopfnicken direkt nachdem der Spieler "Ich helfe dir." gewählt hat.
- **Angriffs-Animation vor dem Kampf** — NPC zieht Waffe und wartet auf Montage-Ende bevor Dialog-Exit.
- **Überraschungsreaktion** — NPC schreckt zurück (Fire-and-Forget, kein Warten), Dialog läuft sofort weiter.
- **Ritual-Szene** — NPC vollführt eine lange Beschwörungsanimation, Dialog-Text erscheint erst danach.

---

> 📸 **Bild-Platzhalter:** `play-animation-node.png` — Node "Play Animation" im MayDialogue-Graphen.
> *Setup:* Node allein, Title-Bar "Play Animation" (Kategorie-Farbe: orange/animation). Subtitle zeigt: `AnimationTargetTag = Dialogue.Speaker.NPC`, `Montage = AM_Nod`. Input-Pin links, Output-Pin rechts.

---

## Properties

| Property | Typ | Beschreibung |
|---|---|---|
| `Montage` | `UAnimMontage*` | Das Montage-Asset, das gespielt wird. |
| `AnimationTargetTag` | `FGameplayTag` | Tag des Participants, dessen AnimInstance die Montage bekommt. |
| `PlayRate` | `float` | Wiedergabe-Geschwindigkeit. `1.0` = normal. Minimum: 0.01. |
| `StartSection` | `FName` | Start-Section innerhalb der Montage. Leer = ab Anfang. |
| `bWaitForMontageEnd` | `bool` | Wenn `true`: Dialog pausiert bis `OnMontageEnded` feuert. Wenn `false`: Fire-and-Forget. |

---

> 📸 **Bild-Platzhalter:** `play-animation-details.png` — Details-Panel mit gefüllten Werten.
> *Setup:* Node auswählen. Im Details-Panel sichtbar: `Montage = AM_GuardReact`, `AnimationTargetTag = Dialogue.Speaker.Guard`, `PlayRate = 1.0`, `StartSection = (leer)`, `bWaitForMontageEnd = true`.

> 📸 **Bild-Platzhalter:** `play-animation-ingame-npc.png` — NPC in der Montage-Pose im PIE-Viewport.
> *Setup:* PIE starten, Dialog bis zum PlayAnimation-Schritt durchlaufen. Screenshot des NPC während der Montage (Kopfnick-Pose oder Ausweich-Pose). Kamera sollte den NPC zeigen, HUD sichtbar mit aktivem Dialogtext.

---

## Action-Node oder SideEffect-Sub-Node?

Wenn die Animation der **Hauptpunkt des Schrittes** ist (z.B. NPC nickt feierlich als Antwort auf eine Wahl), nimm den Action-Node. Wenn die Animation nur ein Begleiteffekt einer SayLine ist (NPC gestikuliert beim Reden), hänge sie als SideEffect-Pill an die SayLine.

---

## Beispiel: Geste + Reaktionszeile

```text
[PlayerChoice: "Ich werde euch helfen."]
  │
  ▼
[PlayAnimation: AM_NodAgreement, Target=NPC, bWaitForMontageEnd=false]
  │
  ▼
[SayLine: NPC "Gut. Dann beeile dich."]
```

Für ein wartendes Beispiel (Ritual-Szene):

```text
[SayLine: NPC "Beobachte genau."]
  │
  ▼
[PlayAnimation: AM_Ritual, Target=Shaman, bWaitForMontageEnd=true]
  │
  ▼
[SayLine: NPC "So ist es vollbracht."]
```

> 📸 **Bild-Platzhalter:** `play-animation-example-graph.png` — Graphausschnitt des Ritual-Beispiels.
> *Setup:* Drei Nodes von links nach rechts: SayLine (NPC) → PlayAnimation (bWaitForMontageEnd=true im Subtitle sichtbar) → SayLine (NPC). Pins verbunden.

---

## Fallstricke

{% hint style="danger" %}
**Bekannter Bug:** Wenn der Dialog abbricht während eine wartende Montage (`bWaitForMontageEnd=true`) noch läuft, kann der `OnMontageEnded`-Delegate nicht korrekt aufgeräumt werden. Workaround: Nutze `bWaitForMontageEnd=false` auf Abläufen, die abgebrochen werden können (z.B. vorzeitiger Dialog-Exit durch Spieler).
{% endhint %}

{% hint style="warning" %}
Der Validator warnt, wenn `bWaitForMontageEnd=true` gesetzt ist, aber kein Output-Pin verbunden ist. In dem Fall würde der Dialog ewig warten.
{% endhint %}

- `AnimationTargetTag` muss einem registrierten Participant entsprechen — sonst kein Play, Log-Warning.
- `StartSection` muss eine gültige Section-Name der Montage sein — sonst spielt die Montage ab Anfang.
