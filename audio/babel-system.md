---
description: Prozedurale Nonsense-Stimmen wenn kein Voice-Asset vorhanden ist – als Entwicklungs-Platzhalter oder dauerhafter Klang-Stil.
---

# Babel-System

## Was ist Babel?

Babel erzeugt automatisch akustisches Begleit-Material für jede Dialogue-Zeile, die kein Voice-Asset hat. Das Ergebnis klingt wie Sprache, ist aber kein echtes Wort – "Nonsense Speech".

**Zwei typische Einsatzfälle:**

- **Platzhalter in der Entwicklung:** Babel überbrückt, bis echte Voice-Aufnahmen vorliegen. Kein Stille im PIE-Test, keine fehlenden Audio-Flags – der Dialog klingt schon nach etwas.
- **Dauerhafter Stil:** Babel als bewusstes Design-Mittel. Animal-Crossing-Blips, Fears-to-Fathom-Murmeln, unheimliche Kreatur-Phoneme – Babel ist darauf ausgelegt, im Fertigen Spiel zu bleiben, wenn du willst.

## Wann greift Babel?

Babel is enabled by default. Es greift für eine SayLine, wenn:

1. Die `DialogueVoice`-Map keinen Eintrag für die aktuelle Culture hat **und**
2. Kein Default-Voice-Asset vorhanden ist

Globaler Schalter in den Project Settings:

> 📸 **Bild-Platzhalter:** `babel-project-settings.png` — Project Settings mit MayDialogue-Audio-Sektion, Fokus auf bEnableBabelVoice und DefaultBabelProfile.
> *Setup:* Editor → Edit → Project Settings → Plugins → MayDialogue → Audio. Sichtbar: `bEnableBabelVoice = true` (Checkbox angehakt), `DefaultBabelProfile` als Asset-Slot mit einem zugewiesenen `BP_Babel_Default`-Asset. Roter Pfeil auf beide Felder.

| Setting | Wirkung |
|---|---|
| `bEnableBabelVoice` | Babel global an/aus (Default: `true`) |
| `DefaultBabelProfile` | Profil wenn kein Sprecher-Profil gesetzt |
| `BabelEngine` | Synthesis engine: `Granular` (recommended, sample-pool based) or `BiquadLegacy` (original sinusoidal DSP path). Configurable via `EMayDialogueBabelEngine`. |

{% hint style="info" %}
`BabelEngine = Granular` uses the pre-recorded sample pool in `Content/DefaultBlips/Sounds/` for higher audio quality (Fears-to-Fathom / Animal-Crossing style). `BiquadLegacy` preserves the original procedural DSP path.
{% endhint %}

## Zwei Synthese-Modi

Babel kann auf zwei verschiedene Weisen klingen:

| Modus | Wie es klingt | Typischer Einsatz |
|---|---|---|
| **BlipPerCharacter** | Pro enthülltem Zeichen ein kurzer Sound ("bip bip bip") | VN-Blips, Animal-Crossing-Stil, freundliche NPCs |
| **PhonemeBase** | Sinusoidale Töne basierend auf Vokal/Konsonant-Typ | Abstrakte Kreaturen, unheimliche Stimmen, nicht-menschliche Entitäten |

## Zwei Sync-Modi

Wie Babel mit dem Typewriter-Text synchronisiert:

| Modus | Wie | Wann |
|---|---|---|
| **TypewriterSync** | Reagiert auf jeden enthüllten Buchstaben des Typewriters | Standard – präzise Synchronisation, jeder Blip mit dem richtigen Zeichen |
| **Continuous** | Interner Timer, unabhängig vom Typewriter | Wenn du keinen Typewriter nutzt oder Kreatur-Groll simulierst |

Der übliche Case ist **BlipPerCharacter + TypewriterSync**: ein Blip pro Zeichen, synchron mit der Textanimation.

## Babel pro Sprecher

Jeder Sprecher kann ein **eigenes Babel-Profil** haben. Das ist der Kern der Babel-Flexibilität:

- Der Wächter hat kurze, tiefe Blips
- Das Kind hat hohe, schnelle Blips
- Der Geist hat PhonemeBase mit niedriger Frequenz
- Der Monster-NPC hat keine Blips, sondern ein kontinuierliches Phoneme-Groll-Muster

Babel-Profil am Sprecher setzen: Speakers-Panel → Sprecher aufklappen → `BabelProfile` → Asset auswählen.

> 📸 **Bild-Platzhalter:** `babel-speaker-profile-assignment.png` — Speakers-Panel mit zwei verschiedenen Sprechern, jeder mit eigenem BabelProfile-Asset.
> *Setup:* Dialog-Asset öffnen, Speakers-Panel. Zwei Sprecher ausgeklappt sichtbar: "Guard" mit `BabelProfile = BP_Babel_Guard`, "Ghost" mit `BabelProfile = BP_Babel_Ghost`. Asset-Icons gut erkennbar, unterschiedliche Asset-Namen zeigen Individualität.

## Babel-Profile anlegen

Ein Babel-Profil ist ein DataAsset:

1. Content Browser → Rechtsklick → **DataAsset**
2. Parent-Class: `UMayDialogueBabelProfile`
3. Benennen (z.B. `BP_Babel_Ghost`)
4. Öffnen und Parameter setzen

> 📸 **Bild-Platzhalter:** `babel-profile-asset-open.png` — Geöffnetes BabelProfile-Asset im Details-Panel mit ausgefüllten Werten.
> *Setup:* Content Browser → DataAsset `BP_Babel_Ghost` doppelklicken. Details-Panel zeigt alle Properties: `BabelMode = PhonemeBase`, `SyncMode = Continuous`, `PhonemeBaseFrequency = 120`, `PhonemeFrequencyRange = 40`, `PhonemeDuration = 0.12`, `bApplyProsody = true`, `Volume = 0.5`. Roter Pfeil auf `BabelMode`.

Details zu allen Properties: [Babel-Profile](babel-profiles.md).

## Preview-Runner

Im Preview-Runner wird Babel automatisch **2D** gespielt – korrekt für den Editor-Test ohne Level-Setup. Im PIE mit einem Sprecher-Actor spielt Babel 3D (sofern kein `Force2D`-Override aktiv ist).

## Widget-Integration

**Monolithisches Widget (Legacy):** `BabelSynth::OnCharacterRevealed` wird automatisch aus dem Typewriter-Text gefeuert – kein Setup nötig.

**UMG-Komponenten-Pfad:** Im Blueprint musst du `TextWidget->OnCharacterRevealed` manuell an `BabelSynth->OnCharacterRevealed` binden. Sieh dazu [UI-Architektur](../ui/umg-architecture.md).

{% hint style="info" %}
**Qualitäts-Anspruch:** Babel ist nicht nur ein Debug-Platzhalter. Mit eigenen BlipSounds, PitchVariation und PunctuationPause kann das Ergebnis so gut klingen, dass du es im fertigen Spiel behalten willst.
{% endhint %}

{% hint style="warning" %}
**Continuous Mode im Komponenten-Pfad:** `Tick` läuft nicht automatisch – du musst es im Blueprint deines Widgets manuell aufrufen. Im SayLine-Fallback-Pfad (ohne Widget) tickt Babel selbst.
{% endhint %}
