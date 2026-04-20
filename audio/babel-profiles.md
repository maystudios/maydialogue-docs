# Babel-Profile

Ein `UMayDialogueBabelProfile` ist ein **DataAsset**, das die Synthese-Parameter für einen Sprecher bündelt.

## Erstellen

1. Content Browser → Rechtsklick → **DataAsset**.
2. Parent-Class: `UMayDialogueBabelProfile`.
3. Benennen, z.B. `BP_Babel_Ghost`.

## Eigenschaften

### Mode

| Property | Typ | Default |
| --- | --- | --- |
| `BabelMode` | `EMayDialogueBabelMode` | `BlipPerCharacter` |
| `SyncMode` | `EMayDialogueBabelSyncMode` | `TypewriterSync` |

### Blip-Mode-Parameter

| Property | Typ | Default | Wirkung |
| --- | --- | --- | --- |
| `BlipSounds` | `TArray<USoundBase*>` | leer | Pool von Samples; Char-Index mod Array-Size wählt. |
| `BasePitch` | float | 1.0 | Basis-Multiplikator. |
| `PitchVariation` | float | 0.15 | ±-Range für Zufall pro Blip. |
| `bVaryPitchByVowel` | bool | true | Vokale bekommen `VowelPitchMultiplier`. |
| `VowelPitchMultiplier` | float | 1.2 | Pitch-Scale für Vokale. |
| `bSkipSpaces` | bool | true | Keine Blips bei Leerzeichen. |
| `bSkipRepeatedChars` | bool | false | Kein Blip, wenn Char gleich dem vorherigen. |

### Phoneme-Mode-Parameter

| Property | Typ | Default | Wirkung |
| --- | --- | --- | --- |
| `PhonemeBaseFrequency` | float | 200 Hz | Grundfrequenz. |
| `PhonemeFrequencyRange` | float | 100 Hz | Variation pro Phonem. |
| `PhonemeDuration` | float | 0.06 s | Ton-Länge. |
| `bApplyProsody` | bool | true | Frequenz je Zeichen-Typ anpassen (Vokal/Konsonant/Zischlaut/Nasal). |

### Timing

| Property | Typ | Default | Wirkung |
| --- | --- | --- | --- |
| `PunctuationPause` | float | 0.15 s | Extra-Pause nach `.`, `!`, `?`. |
| `CommaPause` | float | 0.08 s | Extra-Pause nach `,`. |

### Volume

| Property | Typ | Default | Wirkung |
| --- | --- | --- | --- |
| `Volume` | float | 0.7 | Master-Volumen für alle Blips/Phoneme des Profils. |
| `bUseProceduralDefaults` | bool | true | Wenn `BlipSounds` leer: generiere Procedural-Blips on-the-fly. |

## Beispiel-Profile

### Freundlicher NPC – Animal-Crossing-Style

```
BabelMode          = BlipPerCharacter
SyncMode           = TypewriterSync
BlipSounds         = [BP_Sample_01, BP_Sample_02, BP_Sample_03]  (drei weiche „boop"-Samples)
BasePitch          = 1.1
PitchVariation     = 0.3
bVaryPitchByVowel  = true
VowelPitchMultiplier = 1.3
bSkipSpaces        = true
PunctuationPause   = 0.25
Volume             = 0.6
```

### Unheimlicher Geist – Phoneme-Stil

```
BabelMode              = PhonemeBase
SyncMode               = Continuous
PhonemeBaseFrequency   = 120
PhonemeFrequencyRange  = 40
PhonemeDuration        = 0.12
bApplyProsody          = true
Volume                 = 0.5
```

### Neutral – Fallback für Debug

```
BabelMode               = BlipPerCharacter
BlipSounds              = (leer)
bUseProceduralDefaults  = true  // generiert einfache Sinus-Piepser
Volume                  = 0.5
```

## Zuweisen an einen Sprecher

Im Speakers-Panel des Dialog-Assets: im Sprecher-Eintrag das Feld **BabelProfile** auf das Asset setzen. Alle SayLines dieses Sprechers nutzen nun das Profil (sofern kein Voice-Asset vorhanden ist).

## Globaler Fallback

In [Project Settings](../getting-started/project-settings.md): `DefaultBabelProfile`. Wird genutzt, wenn weder Speaker-Profile noch Voice-Asset vorhanden sind.

## Designer-Tipps

* **Samples in leicht unterschiedlichen Tonhöhen** aufnehmen (3-5 Varianten reichen). Babel verteilt sie per Char-Index, das erzeugt Variation.
* **BasePitch und Variation gemeinsam tunen**: hohe Variation = lebendig, niedrige = monoton/unheimlich.
* **Volume bewusst unter 1.0** halten – Babel soll nicht lauter sein als echte Voices.
* **Keine langen Samples** (< 100 ms) – sonst überlagern sich die Blips beim Typewriter.
