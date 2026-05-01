---
description: Kamera-Shake auf der Spielerkamera auslösen — sofort, räumlich oder global.
---

# Camera Shake

Spielt einen UE-Camera-Shake auf der Spielerkamera ab und gibt sofort Advance zurück. Der Dialog läuft parallel weiter — der Shake überlagert sich mit dem laufenden Dialog ohne ihn zu blockieren.

## Wann nutzen

- **Horror-Jump-Scare** — Ein Monster erscheint, bevor der NPC reagiert. Shake + SayLine gleichzeitig erlebt.
- **Explosion in der Nähe** — Während des Gesprächs detoniert etwas; der Spieler spürt die Druckwelle.
- **Räumlich begrenzter Effekt** — NPC steht nah an einer Maschine: Shake nur wenn Spieler nahe genug ist (`SpatialRadius`).
- **Dramatischer Enthüllungs-Moment** — Starker Shake unmittelbar bevor ein wichtiger Dialog-Satz fällt.

---

> 📸 **Bild-Platzhalter:** `camera-shake-node.png` — Node "Camera Shake" im MayDialogue-Graphen.
> *Setup:* Node allein sichtbar, Title-Bar beschriftet mit "Camera Shake", blau-grau (Kamera-Kategorie), Input-Pin links / Output-Pin rechts. Im Subtitle: `ShakeClass = BP_ShakeHeavy`, `Scale = 2.0`.

---

## Properties

| Property | Typ | Beschreibung |
|---|---|---|
| `ShakeClass` | `TSubclassOf<UCameraShakeBase>` | Beliebige UE-CameraShake-Klasse (Blueprint oder C++). |
| `Scale` | `float` | Shake-Intensität. `1.0` = Normalgröße, `0.0` = kein Effekt. Minimum: 0. |
| `SpatialRadius` | `float` | Wirkungsradius in cm. `0` = global (immer). Wenn > 0: Shake spielt nur, wenn Spieler nah genug am Epizentrum ist. |
| `EpicenterParticipantTag` | `FGameplayTag` | Participant, dessen Position als Epizentrum gilt. Nur aktiv wenn `SpatialRadius > 0`. |
| `FalloffInnerRadius` | `float` | Innerhalb dieses Radius volle Intensität. Zwischen Inner und Outer: lineare Abschwächung. Nur aktiv wenn `SpatialRadius > 0`. |

---

> 📸 **Bild-Platzhalter:** `camera-shake-details.png` — Details-Panel mit räumlichem Setup.
> *Setup:* Node auswählen. Im Details-Panel sichtbar: `ShakeClass = BP_ShakeMedium`, `Scale = 1.5`, `SpatialRadius = 500`, `EpicenterParticipantTag = Dialogue.Speaker.Monster`, `FalloffInnerRadius = 200`.

---

## Action-Node oder SideEffect-Sub-Node?

Wenn der Shake **der dramatische Hauptpunkt** dieses Schritts ist (z.B. der Erschreck-Moment), nimm den Action-Node. Wenn der Shake nur ein subtiler Begleiteffekt einer SayLine ist (leichtes Grummeln beim Sprechen), hänge ihn als SideEffect-Pill an die SayLine.

---

## Beispiel: Jump-Scare-Sequenz

```text
[SayLine: Spieler "Ich höre nichts mehr..."]
  │
  ▼
[CameraShake: ShakeClass=BP_ShakeHeavy, Scale=2.0]
  │
  ▼
[SayLine: Monster "JETZT BIN ICH DA."]
```

> 📸 **Bild-Platzhalter:** `camera-shake-example-graph.png` — Graphausschnitt der obigen Sequenz.
> *Setup:* Drei Nodes von links nach rechts: SayLine (Player) → CameraShake → SayLine (Monster). Alle Pins verbunden. CameraShake-Node zeigt im Subtitle: `Scale = 2.0`.

> 📸 **Bild-Platzhalter:** `camera-shake-ingame-moment.png` — Screenshot des laufenden PIE im Moment des Shakes.
> *Hinweis für den Screenshot:* PIE starten, bis zum CameraShake-Schritt springen. Bild zeigt leicht verwackelte/verschobene Kameraansicht mit dem HUD — sichtbares "Shake"-Artefakt auf dem Frame. Optional: vorher/nachher Frame nebeneinander.

---

## Fallstricke

{% hint style="info" %}
Der Node ist **immer Pass-through** — kein `bWaitForShakeEnd`. Wenn du nach dem Shake eine SayLine willst, die erst erscheint wenn sich die Kamera beruhigt hat, baue eine kurze `Wait`-Node dazwischen.
{% endhint %}

- `SpatialRadius = 0` bedeutet globaler Shake (kein räumliches Falloff, immer abgespielt).
- `EpicenterParticipantTag` muss einem registrierten Participant entsprechen — sonst wird auf Instigator-Position zurückgefallen.
- Shake und CameraFocus überlagern sich ohne Konflikte — der Blend läuft weiter, der Shake legt sich obendrauf.
