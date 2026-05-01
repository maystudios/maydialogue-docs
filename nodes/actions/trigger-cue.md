---
description: Einen GameplayCue one-shot feuern — Partikel, SFX, UI-Flash, kombiniert und repliziert.
---

# Trigger Cue

Feuert einen `GameplayCue` als one-shot über den `AbilitySystemComponent`. Der Cue läuft cosmetically — Partikel, Sound, UI-Effekt — repliziert via GAS auf alle Clients. Pass-through, kein Warten.

## Wann nutzen

- **Fluch-Visualeffekt** — NPC spricht Fluch aus, Partikel + Sound + UI-Flash erscheinen simultan am Spieler.
- **Heilungs-Glow** — Heilerin legt Hand an, `GameplayCue.Dialog.Heal` blinkt grün am NPC.
- **Magischer Treffer** — Zauberer schlägt in die Luft, Cue trifft Spieler (`bTriggerOnInstigator=true`).
- **Multiplayer-sicherer Cosmetic** — Effekt muss auf allen Clients gleichzeitig sichtbar sein (kein PlaySound-Workaround nötig).

---

> 📸 **Bild-Platzhalter:** `trigger-cue-node.png` — Node "Trigger Cue" im MayDialogue-Graphen.
> *Setup:* Node allein, Title-Bar "Trigger Cue" (Kategorie-Farbe: lila/GAS). Subtitle zeigt: `GameplayCue.Dialog.Curse → Target`. Input- und Output-Pin sichtbar.

---

## Properties

| Property | Typ | Beschreibung |
|---|---|---|
| `CueTag` | `FGameplayTag` | Tag des GameplayCue. Muss unter `GameplayCue.*` liegen. |
| `bTriggerOnInstigator` | `bool` | `true` = Cue läuft am Spieler-ASC. `false` = Cue läuft am Ziel-NPC-ASC. Default: `true`. |

---

> 📸 **Bild-Platzhalter:** `trigger-cue-details.png` — Details-Panel mit NPC-Target-Setup.
> *Setup:* Node auswählen. Im Details-Panel sichtbar: `CueTag = GameplayCue.Dialog.Curse`, `bTriggerOnInstigator = false`.

---

## Action-Node oder SideEffect-Sub-Node?

Wenn der Cue-Effekt der **dramatische Hauptmoment** dieses Graph-Schrittes ist (der Fluch-Moment ist bewusst als eigener Schritt inszeniert), nimm den Action-Node. Wenn der Cue nur ein stiller cosmetic Begleiteffekt einer SayLine ist, hänge ihn als SideEffect-Pill an die SayLine.

---

## TriggerCue vs. PlaySound vs. CameraShake

| | Trigger Cue | Play Sound | Camera Shake |
|---|---|---|---|
| System | GAS (`ExecuteGameplayCue`) | UE-Audio direkt | UE-Camera |
| Repliziert | Ja, via GAS | Nein | Ja (`ClientStartCameraShake`) |
| Typischer Inhalt | Partikel + Sound + UI kombiniert | Nur Sound | Nur Kamera-Shake |
| Läuft am | Target-Actor / Instigator | World-Position oder 2D | Spielerkamera |

Wenn du nur einen Sound brauchst: [Play Sound](play-sound.md). Wenn du nur Kamera-Shake brauchst: [Camera Shake](camera-shake.md). Wenn du Partikel + Sound + UI-Flash kombiniert und repliziert brauchst: Trigger Cue.

---

## Beispiel: Fluch-Szene

```text
[SayLine: NPC "Ich verfluche dich!"]
  │
  ▼
[TriggerCue: CueTag=GameplayCue.Dialog.Curse, bTriggerOnInstigator=false]
  │
  ▼
[SayLine: NPC "Mögen die Götter dir nicht mehr folgen."]
```

> 📸 **Bild-Platzhalter:** `trigger-cue-example-graph.png` — Graphausschnitt der Fluch-Szene.
> *Setup:* Drei Nodes: SayLine (NPC) → TriggerCue (GameplayCue.Dialog.Curse, "→ Target" im Subtitle) → SayLine (NPC). Alle Pins verbunden.

---

## Fallstricke

{% hint style="warning" %}
`CueTag` muss unter `GameplayCue.*` liegen — andere Tags-Hierarchien werden vom GAS-Routing nicht erkannt. Ein falsches Prefix führt zu einem stillen No-Op ohne Fehler.
{% endhint %}

{% hint style="info" %}
Für **one-shot Effekte** nutze `GameplayCueNotify_Static` als Cue-Klasse (kein Actor, kein persistenter Zustand). Persistente Cues (`GameplayCueNotify_Actor`) müssen explizit per `Add`/`Remove`-Mechanismus gemanagt werden — Trigger Cue allein reicht dafür nicht.
{% endhint %}

- Der Cue läuft am **Target-Actor-Ursprung**, nicht im Kamera-Space — Partikel erscheinen am NPC, nicht vor der Kamera.
- Diese Aktion ist Teil der GAS-Integration und ohne Zusatzschritte nutzbar. Sie greift auf das Gameplay Ability System zu — stelle sicher, dass deine Charaktere eine `AbilitySystemComponent` haben.
- Ziel muss einen `UAbilitySystemComponent` haben — sonst No-Op + Log-Warning.
