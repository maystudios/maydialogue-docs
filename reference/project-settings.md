# Projekt-Einstellungen (Referenz)

Kompakt-Referenz von `UMayDialogueSettings`. Für die erzählende Erklärung siehe [Getting Started → Projekt-Einstellungen](../getting-started/project-settings.md). Hier geht es nur um die **schnelle Nachschlagbarkeit**.

* **Klasse**: `UMayDialogueSettings` (`UDeveloperSettings`)
* **Config-Category**: `Game`
* **Config-File**: `DefaultGame.ini`
* **Section**: `/Script/MayDialogue.MayDialogueSettings`
* **UI-Pfad im Editor**: *Edit → Project Settings → Plugins → MayDialogue*

## Widget

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `DefaultDialogueWidgetClass` | `TSoftClassPtr<UMayDialogueWidget>` | *(keins)* | UMG-Widget für den Dialog-Overlay. Leer → Slate-Fallback. |
| `bUseSlateDialogueWidget` | `bool` | `true` | Slate-Debug-Widget einblenden, solange `DefaultDialogueWidgetClass` leer ist. |
| `PanelBlurStrength` | `float` | `4.0` | Blur-Stärke der Slate-Panels. Nur wirksam im Slate-Debug-Widget. |

## UMG-Komponenten-Defaults

Fallbacks für den komponenten-basierten UMG-Workflow. Falls `DefaultDialogueWidgetClass` gesetzt ist und `BindWidget`-Slots anbietet, haben die Bindings Vorrang.

| Property | Typ |
| --- | --- |
| `DefaultDialogFrameClass` | `TSoftClassPtr<UMayDialogueWidget_DialogFrame>` |
| `DefaultSpeakerWidgetClass` | `TSoftClassPtr<UMayDialogueWidget_Speaker>` |
| `DefaultTextWidgetClass` | `TSoftClassPtr<UMayDialogueWidget_Text>` |
| `DefaultChoiceButtonClass` | `TSoftClassPtr<UMayDialogueWidget_ChoiceButton>` |
| `DefaultChoiceListClass` | `TSoftClassPtr<UMayDialogueWidget_ChoiceList>` |
| `DefaultSkipButtonClass` | `TSoftClassPtr<UMayDialogueWidget_SkipButton>` |

## Dialog-Defaults

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `DefaultAdvanceMode` | `EMayDialogueAdvanceMode` | `Manual` | Advance-Modus für SayLines ohne Override. |
| `DefaultAutoAdvanceDelay` | `float` | `3.0` | Delay in Sekunden für AdvanceMode `Timer`. |

## Typewriter

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `bEnableTypewriterEffect` | `bool` | `true` | Globaler Typewriter-Schalter. |
| `TypewriterCharsPerSecond` | `float` | `30.0` | Default-Zeichen/Sekunde ohne inline `<speed>`-Tag. |

## Input

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `bAllowSkipTypewriter` | `bool` | `true` | Spieler-Input zippt den Typewriter. |
| `bAllowSkipVoiceLine` | `bool` | `false` | Spieler-Input stoppt laufende Voice und advanced. |
| `bSwitchToUIInputDuringDialogue` | `bool` | `true` | Input-Mode `GameAndUI` für die Dialog-Dauer. |
| `bShowMouseCursorDuringDialogue` | `bool` | `true` | Cursor während des Dialogs einblenden. |

## Audio

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `DefaultSoundClass` | `TSoftObjectPtr<USoundClass>` | *(keins)* | Default-Sound-Class für Voice-Wiedergabe. |
| `DefaultAttenuation` | `TSoftObjectPtr<USoundAttenuation>` | *(keins)* | Default-3D-Attenuation. |
| `bForce2D` | `bool` | `false` | Alle Voice-Wiedergaben in 2D erzwingen. |

## Babel-Voice

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `bEnableBabelVoice` | `bool` | `true` | Babel-Synthese aktivieren, wenn keine Voice gesetzt ist. |
| `DefaultBabelProfile` | `TSoftObjectPtr<UMayDialogueBabelProfile>` | Plugin-Default | Fallback-Profil. |

## Kamera

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `bAutoFocusSpeaker` | `bool` | `false` | Automatischer CameraFocus auf den aktuellen SayLine-Speaker. |
| `DefaultCameraBlendTime` | `float` | `0.75` | Blend-Dauer für CameraFocus ohne explizite Angabe. |

## Programmatischer Zugriff

```cpp
// C++
const UMayDialogueSettings* Settings = GetDefault<UMayDialogueSettings>();
const float Delay = Settings->DefaultAutoAdvanceDelay;

// Mutierendes Setzen (selten; nur in Tools / Migrations):
UMayDialogueSettings* Mutable = GetMutableDefault<UMayDialogueSettings>();
Mutable->bEnableBabelVoice = false;
Mutable->SaveConfig();
```

```
// Blueprint
[Get Default Object] (Class: MayDialogueSettings) → Break Struct / Get Property
```

## Konfigurations-Persistenz

`UDeveloperSettings` schreibt mit `SaveConfig()` nach `DefaultGame.ini` in die Section `/Script/MayDialogue.MayDialogueSettings`. Beispiel:

```ini
[/Script/MayDialogue.MayDialogueSettings]
bEnableTypewriterEffect=True
TypewriterCharsPerSecond=42.0
bEnableBabelVoice=True
DefaultCameraBlendTime=0.5
```

## Shipping-Verhalten

* Alle Werte sind in Shipping **eingefroren** – keine Runtime-Mutation geplant.
* `TSoftObjectPtr` / `TSoftClassPtr` werden beim ersten Dialog-Start lazy resolved; Performance-Impact ist eine einmalige Load-Pause.
* Wenn du trotzdem Runtime-Wechsel brauchst (z.B. Player-Preferences für Typewriter-Speed): Leg eine separate `USaveGame`-Struktur an und lies sie vor `StartDialogue`.

## Siehe auch

* [Editor-Einstellungen](editor-settings.md)
* [Getting Started → Projekt-Einstellungen](../getting-started/project-settings.md) – narrativer Walkthrough.
* [UI → UMG-Architektur](../ui/umg-architecture.md) – wie die UMG-Defaults das Widget-Layout steuern.
* [Audio → Babel-System](../audio/babel-system.md) – Babel-Details.
