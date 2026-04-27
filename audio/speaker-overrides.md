---
description: Pro Sprecher eigene SoundClass, Attenuation, 2D/3D-Modus, Volume und Pitch setzen – einmal konfiguriert, überall konsistent.
---

# Speaker-Overrides

## Wann brauche ich das?

Wenn ein Charakter **immer** anders klingen soll als der Rest – unabhängig davon, welche Zeile er sagt. Typische Fälle:

- Der Geist flüstert leise und hallt nach
- Der Narrator spricht nie räumlich, immer direkt ins Ohr
- Das Monster hat eine tiefere Grundtonhöhe
- Der Protagonist ist generell etwas leiser als die NPCs

Speaker-Overrides setzen diese Klang-Identität **einmalig** im Speakers-Panel. Danach gilt sie automatisch für alle SayLines dieses Sprechers – ohne dass du jeden Node einzeln anfassen musst.

## Wo konfigurieren?

Speakers-Panel im Dialog-Asset → Sprecher-Eintrag aufklappen → Bereich "Audio Overrides".

> 📸 **Bild-Platzhalter:** `speaker-overrides-panel.png` — Speakers-Panel mit aufgeklapptem Sprecher-Eintrag und sichtbaren Audio-Override-Feldern.
> *Setup:* Dialog-Asset öffnen, Speakers-Panel links. Sprecher-Eintrag "Ghost" aufklappen. Im aufgeklappten Bereich sichtbar: `AudioModeOverride` (Dropdown), `SoundClassOverride` (Asset-Slot), `AttenuationOverride` (Asset-Slot), `VolumeMultiplier` (float), `PitchMultiplier` (float), `BabelProfile` (Asset-Slot). Alle Felder mit Beispielwerten gefüllt (SC_VoiceGhost, Att_Whisper, 0.7, 0.85).

## Verfügbare Properties

| Property | Typ | Was es tut |
|---|---|---|
| `AudioModeOverride` | Enum | `Default` (vom Plugin), `Spatial3D` (3D erzwingen), `Force2D` (immer 2D) |
| `SoundClassOverride` | USoundClass | Mixer-Klasse für diesen Sprecher (z.B. eigener Reverb-Bus) |
| `AttenuationOverride` | USoundAttenuation | 3D-Radius, Occlusion-Sensibilität für diesen Sprecher |
| `VolumeMultiplier` | float | Multiplikator auf den Plugin-Default-Volume |
| `PitchMultiplier` | float | Multiplikator auf den Plugin-Default-Pitch |
| `BabelProfile` | UMayDialogueBabelProfile | Babel-Profil wenn kein Voice-Asset vorhanden |

## Praxisbeispiele

### Der Geist klingt geisterhaft

```text
SoundClassOverride  = SC_VoiceGhost    ← eigener Reverb-Send-Bus
AttenuationOverride = Att_GhostWhisper ← kurzer Radius, hohe Occlusion-Sens.
PitchMultiplier     = 0.85             ← etwas tiefer als normal
VolumeMultiplier    = 0.7              ← deutlich leiser
```

Alle SayLines des Geistes klingen so – ohne Node-by-Node-Setup.

> 📸 **Bild-Platzhalter:** `speaker-overrides-ghost-filled.png` — Details-Panel des "Ghost"-Sprechers mit ausgefüllten Werten.
> *Setup:* Dialog-Asset öffnen, Ghost-Sprecher aufklappen. Sichtbar im Details-Bereich: `AudioModeOverride = Default`, `SoundClassOverride = SC_VoiceGhost`, `AttenuationOverride = Att_GhostWhisper`, `VolumeMultiplier = 0.7`, `PitchMultiplier = 0.85`. Felder sauber mit roten Pfeil-Annotationen auf die gesetzten Werte zeigen.

### Der Narrator spricht immer 2D

```text
AudioModeOverride   = Force2D
SoundClassOverride  = SC_VoiceNarrator  ← hat Dialog-Ducking
```

Narrator-Zeilen werden nie räumlich positioniert – korrekt für Off-Screen-Erzähler.

### Der Protagonist ist leiser

```text
VolumeMultiplier = 0.9
```

Alle NPCs laufen auf `1.0`, der Player-Character ist minimal zurückgenommen – subtile Mischung.

### Das Monster hat prozedurale Stimme

```text
BabelProfile = BP_Babel_Monster  ← PhonemeBase-Modus, tiefe Frequenz
```

Kein Voice-Asset vorhanden, Babel übernimmt – jede Zeile klingt nach Kreatur-Groll.

## Konsistenz über mehrere Dialog-Assets

Speaker-Overrides leben **im Dialog-Asset**, nicht global. Das bedeutet: wenn du denselben Charakter in zehn verschiedenen Assets nutzt, müssen die Overrides dort jeweils identisch sein.

{% hint style="info" %}
**Tipp:** Halte eine DataTable mit Speaker-Templates (SoundClass, Attenuation, Pitch, Volume, BabelProfile pro Charakter). Copy-Paste die Zeile in neue Dialog-Assets. So bleibt die Klang-Identität konsistent, ohne sie von Hand nachzutippen.
{% endhint %}

> 📸 **Bild-Platzhalter:** `speaker-overrides-datatable.png` — DataTable mit Speaker-Audio-Templates im Content Browser.
> *Setup:* Content Browser, DataTable-Asset öffnen (Row-Struct mit den Audio-Override-Feldern). Sichtbar: mindestens drei Zeilen (Ghost, Narrator, Monster) mit ausgefüllten SoundClass-, Attenuation-, Volume- und Pitch-Werten pro Reihe. Tabellen-Layout gut lesbar.

## Speaker-Override vs. Node-Override

| Situation | Richtige Ebene |
|---|---|
| Charakter klingt **immer** so | Speaker-Override |
| Nur **diese eine Zeile** ist anders | [Node-Override](node-overrides.md) |
| Gilt für eine ganze Szene, mehrere Sprecher | Plugin-Default anpassen oder alternativen Speaker-Eintrag anlegen |

{% hint style="warning" %}
`AttenuationOverride` am Sprecher setzt die Attenuation immer – auch wenn der Actor selbst einen `UAudioComponent` mit eigener Attenuation-Einstellung hätte. Der Sprecher-Override hat Vorrang.
{% endhint %}
