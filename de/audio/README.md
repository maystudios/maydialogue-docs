---
description: Wie MayDialogue Audio abspielt – von der automatischen 3D-Quelle bis zum prozeduralen Babel-Platzhalter.
---

# Audio-System

Wenn ein Charakter im Level steht, kommt seine Stimme direkt aus dem Level – positioniert am Actor, mit Attenuation, Occlusion und Reverb. Du legst keinen einzigen AudioComponent per Hand an.

> 📸 **Bild-Platzhalter:** `audio-overview-diagram.png` — Pfeil-Diagramm der Audio-Hierarchie.
> *Setup:* Grafik erstellen (kein Editor-Screenshot). Von oben nach unten: `[Plugin-Default (Project Settings)]` → Pfeil nach rechts → `[Speaker-Override (Dialog-Asset)]` → Pfeil nach rechts → `[Node-Override (SayLine / PlaySound)]` → Pfeil nach rechts → `[UAudioComponent am Actor]`. Unter dem letzten Kasten zwei Äste: `[3D-Spatial (Standard)]` und `[2D PlaySound (Force2D)]`. Layout horizontal auf weißem Hintergrund, dunkelgraue Pfeile.

## Wie funktioniert Audio in MayDialogue?

### 1. AudioComponent wird automatisch angelegt

Beim ersten SayLine eines Sprechers erzeugt das Plugin einen `UAudioComponent`, befestigt ihn am Actor und behält ihn für die Dauer des Dialogs. Am Dialog-Ende wird er aufgeräumt.

**Du musst nichts vorbereiten.** Kein Component-Slot am Charakter, kein `BeginPlay`-Setup.

### 2. 3D-First

Standard-Modus: der Sound spielt am Actor-Transform. Dreht sich der Spieler weg, klingt der Charakter leiser. Steht eine Wand dazwischen, greift Occlusion.

### 3. Drei-Ebenen-Fallback

Jede Audio-Eigenschaft kann auf drei Ebenen gesetzt werden – von grob nach fein:

| Ebene | Wo eingestellt | Gilt für |
|---|---|---|
| Plugin-Default | Project Settings | Das ganze Projekt |
| Speaker-Override | Speakers-Panel im Dialog-Asset | Alle SayLines dieses Sprechers |
| Node-Override | SayLine / PlaySound im Graph | Genau diese eine Zeile |

Die spezifischere Ebene gewinnt. Details: [Drei-Ebenen-Fallback](three-level-fallback.md).

### 4. Babel als Platzhalter oder dauerhafter Stil

Fehlt ein Voice-Asset für eine Zeile, greift Babel: eine prozedurale Nonsense-Stimmen-Engine. Sie erzeugt entweder Blips pro Zeichen (Animal-Crossing-Style) oder sinusoidale Phoneme (unheimliche Kreatur-Stimme). Babel ist so konfigurierbar, dass du es im Fertigen Spiel behalten kannst, wenn du willst.

> 📸 **Bild-Platzhalter:** `audio-babel-preview-runner.png` — Preview-Runner während ein Dialog mit Babel-Sprecher läuft.
> *Setup:* Dialog-Asset mit einem Sprecher ohne Voice-Asset öffnen. Preview-Runner starten (Tab unten im Editor). SayLine des Babel-Sprechers ist aktiv, Typewriter-Text läuft durch. Sichtbar: Text-Animation, kein Voice-Asset im Node-Details-Panel, Babel-Indikator aktiv.

## Was das Plugin abdeckt

- On-Demand `UAudioComponent` am Sprecher (kein manuelles Setup)
- Voice-Wiedergabe als `USoundBase` (SoundWave, SoundCue, MetaSound)
- Kultur-basierte Voice-Auflösung (eine Map pro SayLine)
- Babel-Synthese (BlipPerCharacter und PhonemeBase)
- Volume-, Pitch-, SoundClass- und Attenuation-Overrides auf drei Ebenen

## Was das Plugin nicht macht

- Keine Voice-Aufnahme, kein TTS, kein DAW-Import
- Keine Audio-Bus-Verwaltung jenseits des Dialog-Moments
- Keine Lipsync-Generierung (MetaHuman-Lipsync liegt außerhalb des Plugin-Scopes)

> 📸 **Bild-Platzhalter:** `audio-sayline-preview-button.png` — SayLine-Node im Dialog-Editor mit aktivem Preview-Button.
> *Setup:* SayLine-Node auswählen, Details-Panel rechts. Sichtbar: `DialogueText`, Voice-Asset-Slot mit einem gesetzten `USoundBase`-Asset, der **Preview**-Button (Play-Symbol) darunter. Im Hintergrund der Graph-Editor mit dem geöffneten Dialog-Asset.

## Kapitel-Überblick

- [Drei-Ebenen-Fallback](three-level-fallback.md) — Plugin / Speaker / Node im Zusammenspiel
- [Speaker-Overrides](speaker-overrides.md) — pro Sprecher individuelle Klang-Identität
- [Node-Overrides](node-overrides.md) — Ausnahmen pro SayLine oder PlaySound
- [Lokalisierung (VoicePerCulture)](localization.md) — pro Kultur ein anderes Voice-Asset
- [Babel-System](babel-system.md) — prozedurale Nonsense-Stimmen
- [Babel-Profile](babel-profiles.md) — Synthese-Parameter pro Sprecher konfigurieren
