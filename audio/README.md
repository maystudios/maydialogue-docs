# Audio-System

MayDialogue setzt Dialog-Audio **automatisch**. Wenn ein Charakter im Level steht, kommt seine Stimme aus dem Level – ohne dass du ein einziges Audio-Component selbst anlegen musst.

## Philosophie

* **3D-First**. Standard-Modus: Voice spielt am Sprecher-Actor, mit UE-Attenuation, Occlusion, Reverb.
* **2D-Fallback**. Globaler Schalter für Visual-Novel-Modus oder UI-Dialoge.
* **Drei-Ebenen-Fallback**. Plugin → Sprecher → Node – feinste Kontrolle an jeder Stelle.
* **Babel als Platzhalter oder Stil-Option**. Prozedurale Nonsense-Stimmen, wenn kein echtes Voice-Asset vorhanden.

## Kapitel-Überblick

* [Drei-Ebenen-Fallback](three-level-fallback.md) – wie Plugin/Speaker/Node zusammenspielen.
* [Speaker-Overrides](speaker-overrides.md) – pro Sprecher individuelle Audio-Eigenheiten.
* [Node-Overrides](node-overrides.md) – feinste Kontrolle pro SayLine / PlaySound.
* [Lokalisierung (VoicePerCulture)](localization.md) – pro Kultur ein Voice-Asset.
* [Babel-System](babel-system.md) – prozedurale Nonsense-Stimmen.
* [Babel-Profile](babel-profiles.md) – Datenstruktur, die Babel pro Sprecher steuert.

## Was MayDialogue NICHT macht

* **Keine Voice-Aufnahme**, keine TTS-Anbindung, kein DAW-Import.
* **Keine Audio-Bus-Verwaltung über das Projekt hinaus** – MayDialogue hängt sich an dein Audio-Mixing an, übernimmt es aber nicht.
* **Keine Lipsync-Generierung** – wenn du MetaHuman + SFX-Lipsync nutzen willst, ist das außerhalb des Plugin-Scopes.

## Was liefern wir out-of-the-box?

* On-Demand **AudioComponent** am Sprecher (wird erst beim ersten SayLine angelegt).
* **Voice-Wiedergabe** als `USoundBase` (SoundWave, SoundCue, MetaSound).
* **Culture-basierte Voice-Auflösung**.
* **Babel-Synthese** (zwei Modi).
* **Volume-/Pitch-/SoundClass-/Attenuation-Overrides** auf drei Ebenen.

## Audio im Editor

Der SayLine-Node hat einen **Preview-Button**, der das Voice-Asset direkt im Editor abspielt. Der [Preview-Runner](../editor/preview-runner.md) spielt gesamte Dialoge inkl. Audio und Babel 2D ab – ohne PIE.
