# Speaker-Overrides

Jeder Sprecher im Asset kann Audio-Einstellungen übersteuern. Diese Overrides gelten für **alle SayLines dieses Sprechers** – konsistente Charakter-Sound-Identität ohne Node-by-Node-Konfiguration.

## Eigenschaften

Verfügbar im [Speakers-Panel](../editor/speakers-panel.md) beim Ausklappen eines Sprechers:

| Property | Typ | Wirkung |
| --- | --- | --- |
| `AudioModeOverride` | `EMayDialogueAudioMode` | `Default` (vom Plugin) / `Spatial3D` / `Force2D`. |
| `SoundClassOverride` | `USoundClass*` | Mixer-Klasse übersteuern. |
| `AttenuationOverride` | `USoundAttenuation*` | 3D-Räumlichkeit übersteuern. |
| `VolumeMultiplier` | float | Sprecher-Grundvolumen. |
| `PitchMultiplier` | float | Sprecher-Grund-Tonhöhe. |
| `BabelProfile` | `UMayDialogueBabelProfile*` | Eigenes Babel-Profil für diesen Sprecher. |

## Einsatzmuster

### Der Geist klingt geisterhaft

* `SoundClassOverride = SC_VoiceGhost` (hat eigenen Reverb-Bus).
* `AttenuationOverride = Att_GhostWhisper` (kurzer Radius, hohe Occlusion-Sensibilität).
* `PitchMultiplier = 0.85`.
* `VolumeMultiplier = 0.7`.

Jeder SayLine des Geistes klingt gleich, ohne dass du es jedem Node einzeln beibringen musst.

### Der Narrator ist immer 2D

* `AudioModeOverride = Force2D`.
* Vielleicht `SoundClassOverride = SC_VoiceNarrator` (mit Dialog-Ducking).

### Der Protagonist ist leiser

* `VolumeMultiplier = 0.9` – falls alle NPCs bei 1.0 laufen und der Player etwas leiser sein soll.

## Konsistenz

Weil alle Sprecher-Einstellungen **im Dialog-Asset** leben, bleibt die Charakter-Identität über verschiedene Szenen konsistent – solange du den Speaker-Sub-Struct per Copy/Paste oder per DataTable synchronisiert.

**Tipp**: Halte eine **DataTable** mit Speaker-Templates (SoundClass, Attenuation, Pitch, Volume, BabelProfile pro Charakter). Copy-Paste in neue Dialog-Assets.

## Wann Speaker-Override vs. Node-Override?

| Szenario | Antwort |
| --- | --- |
| Gilt für den Charakter **immer** | Speaker-Override. |
| Gilt für **nur diese eine Zeile** | Node-Override. |
| Gilt für eine **Szene** (mehrere Sprecher) | Plugin-Default wechseln (aufwändig) oder alternative Speaker mit anderem Override. |

## Anmerkungen

* Der **NodeColor** des Speakers ist rein visuell im Graph – nicht Audio-relevant.
* `AttenuationOverride` am Sprecher setzt **immer**, auch wenn der Sprecher auf einem Actor sitzt, der seine eigene Attenuation per AudioComponent hätte.
