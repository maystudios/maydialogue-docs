---
description: Wie Plugin-Default, Speaker-Override und Node-Override zusammenspielen – und welche Ebene wann gewinnt.
---

# Drei-Ebenen-Fallback

Audio-Einstellungen werden auf drei Ebenen gesetzt. Die spezifischere Ebene gewinnt immer.

> 📸 **Bild-Platzhalter:** `audio-three-level-diagram.png` — Pfeil-Diagramm der drei Ebenen.
> *Setup:* Grafik (kein Editor-Screenshot). Von links nach rechts drei Kästen mit Pfeilen: `[Plugin-Default]` → `[Speaker-Override]` → `[Node-Override]` → `[Effektiver Audio-Config]`. Unter jedem Kasten eine kursive Beschriftung: "Project Settings", "Speakers-Panel im Dialog-Asset", "SayLine / PlaySound-Node". Pfeile in dunkelblau, Kästen hellgrau mit weißem Text.

## Die drei Ebenen auf einen Blick

| Ebene | Wo konfigurieren | Priorität |
|---|---|---|
| **Plugin-Default** | Project Settings → MayDialogue | Niedrigste |
| **Speaker-Override** | Speakers-Panel im Dialog-Asset | Mittel |
| **Node-Override** | SayLine- oder PlaySound-Node | Höchste |

## Ebene 1 — Plugin-Default

Einmal pro Projekt gesetzt. Gilt für alles, das keine Override hat.

Relevante Properties in den Project Settings:

| Property | Bedeutung |
|---|---|
| `DefaultSoundClass` | Mixer-Klasse für alle Dialogue-Voices |
| `DefaultAttenuation` | 3D-Räumlichkeit für alle Sprecher |
| `bForce2D` | Projekt-weit 2D erzwingen (z.B. für Visual-Novel-Modus) |
| `bEnableBabelVoice` | Babel global an/aus |
| `DefaultBabelProfile` | Fallback-Profil wenn kein Sprecher-Profil gesetzt |

> 📸 **Bild-Platzhalter:** `audio-project-settings-panel.png` — Project Settings mit MayDialogue-Audio-Sektion.
> *Setup:* Editor → Edit → Project Settings → Plugins → MayDialogue. Sichtbar: Audio-Kategorie aufgeklappt, alle oben genannten Properties mit ihren Default-Werten. `bEnableBabelVoice` auf `true`, `DefaultSoundClass` und `DefaultAttenuation` als Asset-Referenzen.

## Ebene 2 — Speaker-Override

Gilt für **alle SayLines dieses Sprechers** im gesamten Dialog-Asset, sofern der Node keinen eigenen Override setzt.

Properties im Speakers-Panel:

| Property | Typ | Wirkung |
|---|---|---|
| `AudioModeOverride` | Enum | `Default` / `Spatial3D` / `Force2D` |
| `SoundClassOverride` | SoundClass-Asset | Mixer-Klasse des Sprechers |
| `AttenuationOverride` | Attenuation-Asset | 3D-Räumlichkeit des Sprechers |
| `VolumeMultiplier` | float | Grundvolumen-Skala des Sprechers |
| `PitchMultiplier` | float | Tonhöhe-Skala des Sprechers |
| `BabelProfile` | BabelProfile-Asset | Profil für Babel-Synthese |

## Ebene 3 — Node-Override

Feinste Kontrolle. Gilt nur für diese eine SayLine oder diesen PlaySound-Node.

Auf **SayLine** verfügbar:

| Property | Typ | Wirkung |
|---|---|---|
| `NodeAudioMode` | Enum | `Default` / `Spatial3D` / `Force2D` |

Auf **PlaySound** verfügbar:

| Property | Typ | Wirkung |
|---|---|---|
| `Sound` | SoundBase-Asset | Expliziter Sound für diesen Node |
| `VolumeMultiplier` | float | Volume-Skala für diesen Sound |
| `PitchMultiplier` | float | Pitch-Skala für diesen Sound |
| `bForce2D` | bool | 2D-Wiedergabe erzwingen |

## Auflösungs-Algorithmus

Für jede SayLine läuft das Plugin diesen Ablauf:

1. Start mit Plugin-Defaults
2. Hat der Speaker einen Override? → entsprechende Felder überschreiben
3. Hat der Node einen Override? → noch einmal überschreiben
4. Resultierenden Config an den `UAudioComponent` übergeben

## Volume und Pitch multiplizieren sich

Volume- und Pitch-Multiplier **multiplizieren sich** über die Ebenen – sie ersetzen sich nicht:

```text
Effektiver_Volume = Plugin_Vol × Speaker_Vol × Node_Vol
Effektiver_Pitch  = Plugin_Pitch × Speaker_Pitch × Node_Pitch
```

Wenn der Plugin-Default `1.0`, der Sprecher `0.8` und ein PlaySound-Node `0.5` setzt, ist das Ergebnis `0.4`.

## Beispiel: Geist mit innerem Monolog

```text
Plugin-Default:
  SoundClass  = SC_Voice
  Attenuation = Att_Default
  bForce2D    = false

Speaker "Ghost":
  SoundClassOverride  = SC_VoiceGhost
  AttenuationOverride = Att_Whisper
  PitchMultiplier     = 0.85
  VolumeMultiplier    = 0.7

Node SayLine "Ich bin längst tot.":
  NodeAudioMode = Force2D   ← innerer Monolog, direkt ins Ohr
```

**Ergebnis dieser Zeile:**
- SoundClass = `SC_VoiceGhost` (vom Speaker)
- Attenuation = ignoriert (Force2D aktiv)
- Pitch = `0.85` (vom Speaker)
- Volume = `0.7` (vom Speaker)
- Wiedergabe = 2D (vom Node)

> 📸 **Bild-Platzhalter:** `audio-fallback-ghost-example.png` — Speakers-Panel mit "Ghost"-Eintrag und ausgefüllten Audio-Overrides, daneben die SayLine im Graph mit NodeAudioMode = Force2D.
> *Setup:* Dialog-Asset mit Sprecher "Ghost" öffnen. Speakers-Panel links zeigt den Ghost-Eintrag ausgeklappt: `SoundClassOverride`, `AttenuationOverride`, `PitchMultiplier = 0.85`, `VolumeMultiplier = 0.7`. Im Graph rechts die SayLine "Ich bin längst tot." ausgewählt, Details-Panel zeigt `NodeAudioMode = Force2D`.

## Wann welche Ebene?

| Situation | Richtige Ebene |
|---|---|
| Alle Charaktere sollen denselben Mixer nutzen | Plugin-Default |
| Ein Charakter klingt immer gedämpft / verhallt | Speaker-Override |
| Genau diese eine Zeile ist innerer Monolog | Node-Override |
| Cutscene mit anderem Attenuation-Radius | Speaker-Override mit temporärem Speaker-Eintrag |

{% hint style="info" %}
**Tipp für innere Monologe:** Leg einen eigenen Sprecher-Eintrag `Dialogue.Speaker.InnerVoice` an und setze dort `AudioModeOverride = Force2D`. Alle inneren Monologe bekommen diesen Sprecher – sauber getrennt vom 3D-Charakter.
{% endhint %}
