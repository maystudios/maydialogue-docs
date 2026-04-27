# Say Line

Der SayLine-Node lässt einen Sprecher eine Zeile sagen. Er ist der am häufigsten verwendete Node — jede gesprochene oder geschriebene Dialogzeile ist eine SayLine.

## Wann setze ich ihn ein?

- Immer, wenn ein NPC oder der Spieler-Charakter etwas sagt.
- Für Erzähler-Texte (Sprecher-Tag auf einen dedizierten Narrator-Sprecher setzen).
- Als Reaktion nach einer Spieler-Wahl.
- Mit Voice-Asset für vertonten Dialog, ohne Voice-Asset für rein textuellen Dialog (mit optionalem Babel-Fallback).
- Mit EmotionTags, wenn das UI den emotionalen Zustand des Sprechers darstellen soll.

## Properties

| Property | Typ | Standard | Bedeutung |
| --- | --- | --- | --- |
| `SpeakerTag` | `FGameplayTag` | leer | Welcher Sprecher sagt die Zeile. Muss mit einem Eintrag in der Speakers-Liste des Assets übereinstimmen. |
| `DialogueText` | `FText` | leer | Der angezeigte Text. Unterstützt Rich-Text-Tags (siehe unten). |
| `DialogueVoice` | `USoundBase*` | leer | Primäres Voice-Asset (sprachunabhängig). Leer = kein Voice. |
| `VoicePerCulture` | `TMap<FString, USoundBase*>` | leer | Voice-Assets pro Culture-Key (z.B. `"de"`, `"en"`). Überschreibt `DialogueVoice` für die passende Kultur. |
| `VolumeMultiplier` | `float` | `1.0` | Node-spezifischer Lautstärke-Multiplikator (0.0–4.0). |
| `PitchMultiplier` | `float` | `1.0` | Node-spezifischer Pitch-Multiplikator (0.1–4.0). |
| `NodeAudioMode` | `EMayDialogueAudioMode` | `Default` | `Default` = aus Plugin/Sprecher-Einstellung erben; `Force2D` = immer 2D; `Spatial3D` = immer räumlich. |
| `EmotionTags` | `FGameplayTagContainer` | leer | Tags für Emotion/Intensität/Szene, die das UI nutzen kann (z.B. `Dialogue.Emotion.Angry`). |
| `bUseDefaultAdvanceMode` | `bool` | `true` | Wenn aktiv, gilt der globale Advance-Modus aus den Plugin-Settings. |
| `AdvanceModeOverride` | `EMayDialogueAdvanceMode` | `Manual` | Aktiv nur wenn `bUseDefaultAdvanceMode = false`. Optionen: `Manual`, `Timer`, `AfterVoice`, `AfterAnimation`, `Immediate`. |

{% hint style="info" %}
**Inline-Bearbeitung:** Doppelklick auf den SayLine-Node im Graph öffnet einen Text-Editor direkt auf dem Node — du musst nicht das Details-Panel öffnen.
{% endhint %}

> 📸 **Bild-Platzhalter:** `sayline-node-graph.png` — SayLine-Node im Graph mit Beispielwerten.
> *Setup:* Ein SayLine-Node ausgewählt. Auf dem Node sichtbar: Sprecher-Tag `Dialogue.Speaker.Guard`, Text `"Halt! Wer bist du?"`, EmotionTag-Pill `Dialogue.Emotion.Suspicious`. Input-Pin verbunden mit Entry-Node links, Output-Pin verbunden mit PlayerChoice-Node rechts.

> 📸 **Bild-Platzhalter:** `sayline-details-panel.png` — Details-Panel einer SayLine.
> *Setup:* SayLine auswählen. Im Details-Panel sichtbar: `SpeakerTag = Dialogue.Speaker.Guard`, `DialogueText = "Halt! Wer bist du?"`, `DialogueVoice` (leer), `VolumeMultiplier = 1.0`, `PitchMultiplier = 1.0`, `NodeAudioMode = Default`, `EmotionTags = (Dialogue.Emotion.Suspicious)`, `bUseDefaultAdvanceMode = true`.

## Rich-Text-Tags

Im `DialogueText` sind folgende Inline-Tags erlaubt:

| Tag | Wirkung |
| --- | --- |
| `<pause=0.5>` | 0,5 s Pause im Typewriter. |
| `<speed=2.0>` | Verdoppelt die Typewriter-Geschwindigkeit (reset am Zeilenende). |
| `<shake>…</shake>` | Text zittert. |
| `<wave>…</wave>` | Text wellt. |
| `<color=#FF0000>…</color>` | Farbe. |
| `<b>…</b>` | Fett. |

Details: [UI → Rich-Text-Tags](../../ui/rich-text-tags.md).

## Mini-Beispiel

```text
[Entry]
  │
  ▼
[SayLine: Wächter | "Halt! <b>Wer</b> bist du?" | EmotionTag: Suspicious]
  AdvanceModeOverride: Manual
  │
  ▼
[PlayerChoice]
  ├─ Choice 0 "Ein Freund des Königs."
  └─ Choice 1 "Das geht dich nichts an."
```

> 📸 **Bild-Platzhalter:** `sayline-example-graph.png` — Mini-Graph Entry → SayLine → PlayerChoice.
> *Setup:* Graph mit drei Nodes: `Entry` → `SayLine "Halt! Wer bist du?"` (SpeakerTag Guard, EmotionTag Suspicious) → `PlayerChoice` mit zwei Choice-Pills. Verbindungen: Entry-Output → SayLine-Input, SayLine-Output → PlayerChoice-Input.

## Häufige Fallstricke

- **SpeakerTag leer oder falsch**: Die SayLine wird ohne Sprecher-Daten (kein Portrait, kein Name) angezeigt. Stelle sicher, dass der Tag mit einem Sprecher in der Speakers-Liste übereinstimmt.
- **`bUseDefaultAdvanceMode = true` vergessen**: Wenn du für einzelne Nodes einen anderen Advance-Modus willst, deaktiviere zuerst `bUseDefaultAdvanceMode`, sonst hat `AdvanceModeOverride` keine Wirkung.

## Erweitern

{% hint style="info" %}
**Eigene Variante:** Leite eine Blueprint-Subklasse von `UMayDialogueNode_SayLine` ab und überschreibe `ExecuteClientEffects`, um z.B. einen Lipsync-Controller zu starten oder eine eigene Audio-Engine anzusprechen.
{% endhint %}
