# Say Line

Der **SayLine-Node** ist der Arbeitstier-Node. Er repräsentiert eine Zeile, die ein Sprecher sagt.

## Runtime-Verhalten

`ExecuteNode`:

1. Baut eine `FMayDialogueMessage` (Sprecher-Daten, Text, Voice-Asset, EmotionTags, AdvanceMode).
2. Startet Voice-Wiedergabe (falls Voice-Asset vorhanden) oder Babel-Synthese.
3. Ruft `Instance::ReceiveMessage(Message, NextNodeGuid)`.
4. Die Instance broadcastet `OnMessageReceived` und setzt sich in `WaitingForAdvance`.

Der Dialog advanced, sobald der konfigurierte Advance-Mode zuschlägt.

## Properties

| Property | Typ | Zweck |
| --- | --- | --- |
| `SpeakerTag` | `FGameplayTag` | Tag des Sprechers (matched mit Speakers-Liste + Participant-Tag). |
| `DialogueText` | `FText` | Der Text, inklusive Rich-Text-Tags. |
| `DialogueVoice` | `TMap<FString, USoundBase*>` | Voice-Asset pro Culture-Key. Leer für Babel/Still. |
| `EmotionTags` | `FGameplayTagContainer` | Emotion/Intensity/Szene-Tags. |
| `AdvanceModeOverride` | `EMayDialogueAdvanceMode` | Manual / Timer / AfterVoice / AfterAnimation / Immediate. |
| `AutoAdvanceDelayOverride` | `float` | Bei AdvanceMode=Timer: Wartezeit in Sekunden. |
| `NodeAudioMode` | `EMayDialogueAudioMode` | Default / Spatial3D / Force2D. |

## Pins

* **Input**.
* **Output** – zum nächsten Node.

## Rich-Text-Tags im Text

Im `DialogueText` sind folgende Inline-Tags erlaubt:

| Tag | Wirkung |
| --- | --- |
| `<pause=0.5>` | 0.5 s Pause im Typewriter. |
| `<speed=2.0>` | Verdoppelt die Typewriter-Geschwindigkeit (reset am Zeilenende). |
| `<shake>...</shake>` | Text zittert. |
| `<wave>...</wave>` | Text welt. |
| `<color=#FF0000>...</color>` | Farbe. |
| `<b>...</b>` | Fett. |

Details siehe [UI → Rich-Text-Tags](../../ui/rich-text-tags.md).

## Typisches Pattern

```
[Entry]
  │
  ▼
[SayLine: Wächter | "Halt! <b>Wer</b> bist du?" | EmotionTag: Angry]
  │
  ▼
[PlayerChoice: ...]
```

## Anmerkungen

* **Inline-Text-Edit** im Graph per Doppelklick.
* **Preview-Button** im Graph (kleines Play-Icon) spielt das Voice-Asset direkt – ohne PIE.
* **VolumeMultiplier / PitchMultiplier pro Node** fehlen aktuell (Backlog-Item 11). Als Workaround: am Speaker im Speakers-Panel setzen.
* **bOverride2D pro Node** fehlt aktuell (Backlog-Item 10) – entweder globale Force2D oder Speaker-Override nutzen.
