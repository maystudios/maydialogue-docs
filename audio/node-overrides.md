# Node-Overrides

Node-Overrides sind die **feinste Kontroll-Ebene**. Sie gelten für eine einzige Zeile und übersteuern Plugin- und Speaker-Defaults.

## Verfügbare Properties

### Auf SayLine

| Property | Typ | Verfügbar |
| --- | --- | --- |
| `NodeAudioMode` | `EMayDialogueAudioMode` | Ja (`Default` / `Spatial3D` / `Force2D`). |
| `AdvanceModeOverride` | `EMayDialogueAdvanceMode` | Ja (steuert Advance-Logik, audio-relevant bei `AfterVoice`). |
| `VolumeMultiplier` | float | **Aktuell nicht** – Backlog-Item 11. |
| `PitchMultiplier` | float | **Aktuell nicht** – Backlog-Item 11. |
| `bOverride2D` | bool | **Aktuell nicht** – Backlog-Item 10. |

### Auf PlaySound

| Property | Typ |
| --- | --- |
| `Sound` | `USoundBase*` |
| `VolumeMultiplier` | float |
| `PitchMultiplier` | float |
| `bForce2D` | bool |
| `TargetTag` | `FGameplayTag` (optional) |

## Advance-Mode & Audio

Der Advance-Mode auf einer SayLine beeinflusst, wie der Dialog-Flow mit Audio umgeht:

| Modus | Audio-Verhalten |
| --- | --- |
| `Manual` | Audio läuft, Spieler advanced unabhängig (typisch: Audio bricht bei Advance ab, falls `bAllowSkipVoiceLine=true`). |
| `Timer` | Audio läuft, Dialog advanced nach `AutoAdvanceDelay`. Audio endet ggf. davor oder danach. |
| `AfterVoice` | Dialog advanced **genau**, wenn Voice-Playback endet. |
| `AfterAnimation` | Dialog advanced, wenn eine parallele Montage endet. |
| `Immediate` | Kein Warten. Audio startet trotzdem (aber kann vom nächsten Node abgelöst werden). |

## Typische Szenarien

### Innerer Monolog

Auf einer einzelnen SayLine: `NodeAudioMode = Force2D`, um die Zeile nicht im 3D-Raum zu positionieren, sondern direkt ins Ohr des Spielers.

Workaround bis Backlog-10 gefixt: am Speaker `AudioModeOverride = Force2D` setzen und den Speaker nur für innere Monologe nutzen (z.B. ein separater `Dialogue.Speaker.InnerVoice`-Sprecher).

### Ambient-Akzent

PlaySound-Node zwischen zwei SayLines mit eigenem `VolumeMultiplier = 0.4`, weil der Sound nicht dominieren soll.

### Panischer Schrei

PlaySound-Node mit `PitchMultiplier = 1.3`, um die Dringlichkeit zu erhöhen.

## Wann Node-Override vs. Speaker-Override?

Faustregel: **Wenn eine Audio-Einstellung nur für diese eine Zeile sinnvoll ist → Node-Override.** Alles andere gehört auf den Speaker.

## Anmerkungen

* Fehlende SayLine-Audio-Overrides (VolumeMultiplier, PitchMultiplier, bOverride2D) sind bekannter Backlog. Bis dahin: Speaker-Override-Trick oder eigene Blueprint-Subklasse des SayLine-Nodes.
