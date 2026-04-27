---
description: Einen Non-Voice-Sound abspielen — 2D-Stinger, 3D-Ambient oder SFX-Akzent.
---

# Play Sound

Spielt einen beliebigen `USoundBase` ab (SoundCue, SoundWave oder MetaSound). Dieser Node ist ausschließlich für **Nicht-Sprach-Audio** — Donner, Türknarren, Herzschlag, Musik-Stinger. Sprache gehört in den Voice-Slot der SayLine.

## Wann nutzen

- **Atmosphärischer Akzent** — Donnerschlag zwischen zwei Zeilen, 2D, kein Spatial-Setup nötig.
- **3D-Soundeffekt am NPC** — Kette rasselt, Schwert zieht, Schritte — angehängt an Participant-Position.
- **Musik-Stinger** — Kurzes dramatisches Motiv direkt vor einem Enthüllungs-Satz.
- **Horror-Atmosphäre** — Herzschlag-Loop (Fire-and-Forget), Dialog läuft parallel weiter.

---

> 📸 **Bild-Platzhalter:** `play-sound-node.png` — Node "Play Sound" im MayDialogue-Graphen.
> *Setup:* Node allein, Title-Bar "Play Sound" (Kategorie-Farbe: türkis/Audio). Subtitle zeigt: `Sound = SC_Thunder`, `NodeAudioMode = Force2D`. Input- und Output-Pin sichtbar.

---

## Properties

| Property | Typ | Beschreibung |
|---|---|---|
| `Sound` | `USoundBase*` | Das Sound-Asset. Unterstützt SoundCue, SoundWave und MetaSound. |
| `VolumeMultiplier` | `float` | Lautstärke-Multiplikator. Bereich: 0–4. Default: `1.0`. |
| `PitchMultiplier` | `float` | Tonhöhen-Multiplikator. Bereich: 0.1–4. Default: `1.0`. |
| `NodeAudioMode` | `EMayDialogueAudioMode` | `Default` = Plugin-Setting übernehmen. `Force2D` = immer flat. `Spatial3D` = immer 3D am Instigator/Target. |
| `AttenuationOverride` | `USoundAttenuation*` | Node-spezifisches Attenuation-Preset. Überschreibt Plugin-Default. Nur bei 3D aktiv. |

---

> 📸 **Bild-Platzhalter:** `play-sound-details.png` — Details-Panel mit 3D-Setup.
> *Setup:* Node auswählen. Im Details-Panel sichtbar: `Sound = SC_ChainRattle`, `VolumeMultiplier = 1.2`, `PitchMultiplier = 1.0`, `NodeAudioMode = Spatial3D`, `AttenuationOverride = ATT_NPC_Close`.

---

## Action-Node oder SideEffect-Sub-Node?

Wenn der Sound der **dramatische Hauptpunkt** des Schrittes ist (der Donnerschlag ist bewusst platziert zwischen zwei SayLines als eigener Moment), nimm den Action-Node. Wenn der Sound nur nebenbei einer SayLine zugeordnet ist (NPC klopft auf den Tisch beim Reden), hänge ihn als SideEffect-Pill an die SayLine.

---

## Beispiel: Atmosphärischer Übergang

```text
[SayLine: Spieler "Ich habe ein schlechtes Gefühl..."]
  │
  ▼
[PlaySound: SC_Thunder, NodeAudioMode=Force2D, VolumeMultiplier=1.5]
  │
  ▼
[SayLine: Spieler "...und es wird nur schlimmer."]
```

> 📸 **Bild-Platzhalter:** `play-sound-example-graph.png` — Graphausschnitt des Donner-Beispiels.
> *Setup:* Drei Nodes: SayLine (Spieler) → PlaySound (SC_Thunder, Force2D im Subtitle) → SayLine (Spieler). Alle Pins verbunden.

---

## Fallstricke

{% hint style="info" %}
**Voice-Zeilen gehören nicht hierher.** Voice-Audio kommt in den Voice-Slot der SayLine — nicht in PlaySound. PlaySound ist für alles außer Sprache.
{% endhint %}

- Der Node ist immer **Pass-through** — lange Sounds spielen weiter, der Dialog advanced sofort. Für ein "warten bis Sound fertig"-Pattern gibt es aktuell keinen nativen Mechanismus; nutze einen `Wait`-Node mit geschätzter Dauer oder steuere die Länge via SoundCue.
- `NodeAudioMode = Default` erbt die Plugin-weite `bForce2D`-Einstellung aus den Projekt-Settings.
- Wenn `NodeAudioMode = Spatial3D` und kein Participant-Tag gesetzt, wird die Instigator-Position als Spielort genutzt.
- `AttenuationOverride` hat nur bei effektivem 3D-Modus Wirkung — bei `Force2D` wird es ignoriert.
