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

Ein Babel-Profil ist ein DataAsset. Du kannst es als normales DataAsset anlegen **oder** als Blueprint-Subklasse — `UMayDialogueBabelProfile` ist jetzt `Blueprintable`, d.h. du kannst prozedurale Profile bauen, die zur Laufzeit ihre Parameter ändern (z.B. Affections-Level → Pitch):

**Als DataAsset (Standard):**
1. Content Browser → Rechtsklick → **DataAsset**
2. Parent-Class: `UMayDialogueBabelProfile`
3. Benennen (z.B. `BP_Babel_Ghost`)
4. Öffnen und Parameter setzen

**Als Blueprint-Subklasse (procedural):**
1. Content Browser → Rechtsklick → **Blueprint Class**
2. Parent-Class: `UMayDialogueBabelProfile`
3. Properties in `Construction Script` oder `Event BeginPlay` setzen

> 📸 **Bild-Platzhalter:** `babel-profile-asset-open.png` — Geöffnetes BabelProfile-Asset im Details-Panel mit ausgefüllten Werten.
> *Setup:* Content Browser → DataAsset `BP_Babel_Ghost` doppelklicken. Details-Panel zeigt alle Properties: `BabelMode = PhonemeBase`, `SyncMode = Continuous`, `PhonemeBaseFrequency = 120`, `PhonemeFrequencyRange = 40`, `PhonemeDuration = 0.12`, `bApplyProsody = true`, `Volume = 0.5`. Roter Pfeil auf `BabelMode`.

Details zu allen Properties: [Babel-Profile](babel-profiles.md).

## Preview-Runner

Im Preview-Runner wird Babel automatisch **2D** gespielt – korrekt für den Editor-Test ohne Level-Setup. Im PIE mit einem Sprecher-Actor spielt Babel 3D (sofern kein `Force2D`-Override aktiv ist).

## BabelSynth — BP-Callable Methoden

`UMayDialogueBabelSynth` stellt jetzt folgende Methoden direkt in Blueprint bereit (Kategorie `MayDialogue|Audio`):

| Methode | Art | Beschreibung |
| --- | --- | --- |
| `Is Active` | Pure | Prüft ob der Synth gerade spielt |
| `Stop Speech` | Callable | Aktuell laufende Babel-Ausgabe abbrechen |
| `Set External Volume Multiplier` | Callable | Lautstärke von außen skalieren (z.B. für Audio-Ducking) |
| `Set External Pitch Multiplier` | Callable | Tonhöhe von außen skalieren (z.B. für Stress-Effekte) |

Diese Methoden sind nützlich wenn du Babel-Ausgabe aus einem anderen System heraus steuerst — z.B. Babel stumm schalten wenn ein Cutscene startet, oder Pitch für einen betäubten Sprecher absenken.

---

## Widget-Integration

**SMayDialogueWidget (Slate-Debug-Widget, Standard):** Das eingebaute Slate-Widget verdrahtet `BabelSynth::OnCharacterRevealed` automatisch mit dem Typewriter — kein manuelles Setup nötig. Dieses Widget ist der Standard-Fallback, wenn kein UMG-Widget konfiguriert ist. Mehr dazu: [Slate-Debug-Widget](slate-debug-widget.md).

**UMG-Komponenten-Pfad:** Im Blueprint musst du die Typewriter-Events deines Text-Widgets selbst an den Synth weitergeben. Sieh dazu [UI-Architektur](../ui/umg-architecture.md).

{% hint style="info" %}
**Qualitäts-Anspruch:** Babel ist nicht nur ein Debug-Platzhalter. Mit eigenen BlipSounds, PitchVariation und PunctuationPause kann das Ergebnis so gut klingen, dass du es im fertigen Spiel behalten willst.
{% endhint %}

{% hint style="warning" %}
**Continuous Mode im UMG-Pfad:** Der Synth tickt automatisch über Unreal Engines Tick-System (`FTickableGameObject`) — du musst nichts selbst aufrufen. Im TypewriterSync-Modus musst du jedoch jeden aufgedeckten Buchstaben über C++ an `OnCharacterRevealed` weiterleiten. Im Blueprint kannst du den Synth über die verfügbaren Nodes steuern:

```text
[Event On Character Revealed]  ← Typewriter-Event deines Text-Widgets
    │ Char (int32), CharIndex, TotalChars
    ▼
[Is Active]  ← BabelSynth | MayDialogue|Babel
    │ True
    ▼
[Stop Speech]  ← BabelSynth bei Bedarf (z.B. Skip-Button)
```

Für einfache Projekte ist der SayLine-Fallback-Pfad (ohne Widget) die empfohlene Option — dort startet und stoppt Babel vollautomatisch.
{% endhint %}
