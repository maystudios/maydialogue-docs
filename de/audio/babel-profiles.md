---
description: Ein DataAsset mit allen Synthese-Parametern für einen Sprecher – so klingt jeder Charakter anders, auch ohne Voice-Aufnahmen.
---

# Babel-Profile

Ein `UMayDialogueBabelProfile` bündelt alle Synthese-Parameter für einen Sprecher: Modus, Pitch, Samples, Timing, Volume. Jeder Sprecher kann ein eigenes Profil bekommen.

## Profil anlegen

1. Content Browser → Rechtsklick → **May Dialogue → Babel Voice Profile**
2. Benennen, z.B. `BP_Babel_Ghost`, `BP_Babel_Guard`, `BP_Babel_Child`
3. Doppelklicken und Parameter setzen

## Alle Properties im Überblick

### Modus

| Property | Typ | Default | Wirkung |
|---|---|---|---|
| `BabelMode` | Enum | `BlipPerCharacter` | Synthese-Methode: Blip oder Phonem |
| `SyncMode` | Enum | `TypewriterSync` | Zeitsteuerung: Typewriter-Events oder kontinuierlich |

### Blip-Mode-Parameter

Aktiv wenn `BabelMode = BlipPerCharacter`:

| Property | Typ | Default | Wirkung |
|---|---|---|---|
| `BlipSounds` | Array\<USoundBase\> | leer | Pool von Sample-Assets; Char-Index mod Array-Size wählt das Sample |
| `BasePitch` | float | 1.0 | Basis-Pitch-Multiplikator für alle Blips |
| `PitchVariation` | float | 0.15 | ±-Zufall pro Blip (0.0 = monoton, 1.0 = sehr variabel) |
| `bVaryPitchByVowel` | bool | true | Vokale bekommen `VowelPitchMultiplier` |
| `VowelPitchMultiplier` | float | 1.2 | Pitch-Skala speziell für Vokal-Zeichen |
| `bSkipSpaces` | bool | true | Keine Blips bei Leerzeichen |
| `bSkipRepeatedChars` | bool | false | Kein Blip wenn Zeichen gleich dem vorherigen |

> 📸 **Bild-Platzhalter:** `babel-profiles-blip-params.png` — Geöffnetes BabelProfile-Asset (BlipPerCharacter-Modus) mit ausgefüllten Blip-Parametern.
> *Setup:* DataAsset `BP_Babel_Child` öffnen. Details-Panel: `BabelMode = BlipPerCharacter`, `SyncMode = TypewriterSync`. Darunter Blip-Sektion: `BlipSounds` mit 3 Assets gefüllt (BP_Sample_01, 02, 03), `BasePitch = 1.2`, `PitchVariation = 0.35`, `bVaryPitchByVowel = true`, `VowelPitchMultiplier = 1.4`, `bSkipSpaces = true`. Alle Felder gut lesbar.

### Phoneme-Mode-Parameter

Aktiv wenn `BabelMode = PhonemeBase`:

| Property | Typ | Default | Wirkung |
|---|---|---|---|
| `PhonemeBaseFrequency` | float | 200 Hz | Grundfrequenz der generierten Töne |
| `PhonemeFrequencyRange` | float | 100 Hz | Variation pro Phonem-Typ |
| `PhonemeDuration` | float | 0.06 s | Wie lange jeder generierte Ton klingt |
| `bApplyProsody` | bool | true | Vokal/Konsonant/Zischlaut/Nasal bekommen unterschiedliche Frequenzen |

### Timing

| Property | Typ | Default | Wirkung |
|---|---|---|---|
| `PunctuationPause` | float | 0.15 s | Extra-Pause nach `.` `!` `?` |
| `CommaPause` | float | 0.08 s | Extra-Pause nach `,` |

### Volume

| Property | Typ | Default | Wirkung |
|---|---|---|---|
| `Volume` | float | 0.7 | Master-Volume des Profils (Babel soll nicht lauter sein als echte Voices) |
| `bUseProceduralDefaults` | bool | true | Wenn `BlipSounds` leer: generiere einfache Sinus-Piepser on-the-fly |

## Fertige Beispiel-Profile

### Freundlicher NPC — Animal-Crossing-Stil

```text
BabelMode            = BlipPerCharacter
SyncMode             = TypewriterSync
BlipSounds           = [BP_Sample_Boop_01, BP_Sample_Boop_02, BP_Sample_Boop_03]
BasePitch            = 1.1
PitchVariation       = 0.30
bVaryPitchByVowel    = true
VowelPitchMultiplier = 1.3
bSkipSpaces          = true
PunctuationPause     = 0.25
Volume               = 0.6
```

Klingt lebendig, freundlich, variabel. Drei weiche "boop"-Samples reichen für Variation.

### Unheimlicher Geist — Phoneme-Groll

```text
BabelMode             = PhonemeBase
SyncMode              = Continuous
PhonemeBaseFrequency  = 120
PhonemeFrequencyRange = 40
PhonemeDuration       = 0.12
bApplyProsody         = true
Volume                = 0.5
```

Kein Typewriter-Sync, kontinuierliches Murmeln. Tiefe Frequenz, wenig Variation = bedrohlich.

### Debug-Fallback — Neutral

```text
BabelMode              = BlipPerCharacter
BlipSounds             = (leer)
bUseProceduralDefaults = true
Volume                 = 0.5
```

Generiert einfache Sinus-Piepser. Kein Asset nötig, sofort einsatzbereit.

> 📸 **Bild-Platzhalter:** `babel-profiles-three-examples.png` — Content Browser mit drei Babel-Profile-Assets nebeneinander: BP_Babel_NPC, BP_Babel_Ghost, BP_Babel_Debug.
> *Setup:* Content Browser gefiltert auf `BP_Babel_`. Drei DataAsset-Icons sichtbar: `BP_Babel_NPC`, `BP_Babel_Ghost`, `BP_Babel_Debug`. Tooltips oder Asset-Details zeigen unterschiedliche BabelMode-Werte.

## Profil einem Sprecher zuweisen

Im Speakers-Panel des Dialog-Assets:

1. Sprecher-Eintrag aufklappen
2. Feld `BabelProfile` → Asset-Slot → Profil auswählen

Alle SayLines dieses Sprechers nutzen nun das Profil – sofern kein Voice-Asset für die aktuelle Culture vorhanden ist.

## Globaler Fallback

In den Project Settings: `DefaultBabelProfile`. Wird genutzt wenn:
- Kein Voice-Asset vorhanden
- Kein Sprecher-Profil gesetzt
- `bEnableBabelVoice = true`

Setze hier das neutrale Debug-Profil als Basis.

## Designer-Tipps

{% hint style="success" %}
**Samples in leicht unterschiedlichen Tonhöhen aufnehmen** (3–5 Varianten reichen). Babel verteilt sie per Char-Index – das erzeugt organische Variation ohne Pitch-Shifting-Artefakte.
{% endhint %}

{% hint style="info" %}
**BasePitch und PitchVariation zusammen tunen:**
- Hohe Variation (0.3–0.5) = lebendig, spielerisch
- Niedrige Variation (0.0–0.1) = monoton, unheimlich, roboterhaft
{% endhint %}

{% hint style="warning" %}
**Samples kurz halten** (unter 100 ms). Längere Samples überlagern sich beim Typewriter und klingen matschig.

**Volume bewusst unter 1.0** setzen – Babel soll nie lauter sein als echte Voices, wenn diese später eingesetzt werden.
{% endhint %}
