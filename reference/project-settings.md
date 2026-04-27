---
description: Alle Felder von UMayDialogueSettings — Typen, Defaults, Bedeutung.
---

# Projekt-Einstellungen (Referenz)

Kompakt-Referenz von `UMayDialogueSettings`.

- **Klasse**: `UMayDialogueSettings` (`UDeveloperSettings`)
- **Config-Datei**: `DefaultGame.ini`
- **Section**: `/Script/MayDialogue.MayDialogueSettings`
- **UI-Pfad**: *Edit → Project Settings → Plugins → MayDialogue*

> 📸 **Bild-Platzhalter:** `project-settings-panel.png` — Screenshot des Project-Settings-Panels.
> *Setup:* Editor geöffnet, Edit → Project Settings → Plugins → MayDialogue. Vollständiger Screenshot des Settings-Panels mit allen Kategorien sichtbar: Widget, UMG-Komponenten-Defaults, Dialog-Defaults, Typewriter, Input, Audio, Babel-Voice, Kamera. Roter Pfeil zeigt auf den Navigations-Pfad oben im Settings-Fenster.

---

## Widget

| Property | Typ | Default | Bedeutung |
|---|---|---|---|
| `DefaultDialogueWidgetClass` | `TSoftClassPtr<UMayDialogueWidget>` | *(leer)* | UMG-Widget das als Dialog-Overlay angezeigt wird. Leer → Slate-Debug-Fallback. |
| `bUseSlateDialogueWidget` | `bool` | `true` | Slate-Debug-Widget einblenden solange `DefaultDialogueWidgetClass` leer ist. |
| `PanelBlurStrength` | `float` | `4.0` | Blur-Stärke der Slate-Panels (nur im Slate-Debug-Widget wirksam). |

---

## UMG-Komponenten-Defaults

Fallback-Klassen für den komponenten-basierten UMG-Workflow. Wenn `DefaultDialogueWidgetClass` `BindWidget`-Slots hat, haben diese Vorrang.

| Property | Typ |
|---|---|
| `DefaultDialogFrameClass` | `TSoftClassPtr<UMayDialogueWidget_DialogFrame>` |
| `DefaultSpeakerWidgetClass` | `TSoftClassPtr<UMayDialogueWidget_Speaker>` |
| `DefaultTextWidgetClass` | `TSoftClassPtr<UMayDialogueWidget_Text>` |
| `DefaultChoiceButtonClass` | `TSoftClassPtr<UMayDialogueWidget_ChoiceButton>` |
| `DefaultChoiceListClass` | `TSoftClassPtr<UMayDialogueWidget_ChoiceList>` |
| `DefaultSkipButtonClass` | `TSoftClassPtr<UMayDialogueWidget_SkipButton>` |

---

## Dialog-Defaults

| Property | Typ | Default | Bedeutung |
|---|---|---|---|
| `DefaultAdvanceMode` | `EMayDialogueAdvanceMode` | `Manual` | Advance-Modus für SayLines ohne expliziten Override. |
| `DefaultAutoAdvanceDelay` | `float` | `3.0` | Delay in Sekunden für AdvanceMode `Timer`. |

---

## Typewriter

| Property | Typ | Default | Bedeutung |
|---|---|---|---|
| `bEnableTypewriterEffect` | `bool` | `true` | Globaler Typewriter-Schalter. |
| `TypewriterCharsPerSecond` | `float` | `30.0` | Zeichen pro Sekunde (Standard, ohne `<speed>`-Inline-Tag). |

---

## Input

| Property | Typ | Default | Bedeutung |
|---|---|---|---|
| `bAllowSkipTypewriter` | `bool` | `true` | Spieler-Input zipped den Typewriter sofort auf den vollen Text. |
| `bAllowSkipVoiceLine` | `bool` | `false` | Spieler-Input stoppt laufende Voice und advanced weiter. |
| `bSwitchToUIInputDuringDialogue` | `bool` | `true` | Input-Mode `GameAndUI` für die Dialog-Dauer. |
| `bShowMouseCursorDuringDialogue` | `bool` | `true` | Maus-Cursor während des Dialogs einblenden. |

---

## Audio

| Property | Typ | Default | Bedeutung |
|---|---|---|---|
| `DefaultSoundClass` | `TSoftObjectPtr<USoundClass>` | *(leer)* | Default-Sound-Class für Voice-Wiedergabe. |
| `DefaultAttenuation` | `TSoftObjectPtr<USoundAttenuation>` | *(leer)* | Default-3D-Attenuation für Voice-Audio. |
| `bForce2D` | `bool` | `false` | Alle Voice-Wiedergaben als 2D erzwingen (ignoriert Attenuation). |

---

## Babel-Voice

| Property | Typ | Default | Bedeutung |
|---|---|---|---|
| `bEnableBabelVoice` | `bool` | `true` | Babel-Synthese aktivieren wenn kein Voice-Asset gesetzt ist. |
| `DefaultBabelProfile` | `TSoftObjectPtr<UMayDialogueBabelProfile>` | Plugin-Default | Fallback-Profil wenn der Sprecher kein eigenes Profil hat. |

---

## Kamera

| Property | Typ | Default | Bedeutung |
|---|---|---|---|
| `bAutoFocusSpeaker` | `bool` | `false` | Automatischer CameraFocus auf den aktuellen SayLine-Speaker. |
| `DefaultCameraBlendTime` | `float` | `0.75` | Blend-Dauer in Sekunden für CameraFocus ohne explizite Angabe. |

---

## Programmatischer Zugriff

```cpp
const UMayDialogueSettings* S = GetDefault<UMayDialogueSettings>();
float Delay = S->DefaultAutoAdvanceDelay;
bool bTypewriter = S->bEnableTypewriterEffect;
```

Blueprint-Zugriff:

```text
[Get Class Defaults] (Class: MayDialogueSettings) → Properties lesen
```

---

## Config-Beispiel (DefaultGame.ini)

```ini
[/Script/MayDialogue.MayDialogueSettings]
bEnableTypewriterEffect=True
TypewriterCharsPerSecond=42.0
bEnableBabelVoice=True
DefaultAdvanceMode=Manual
DefaultAutoAdvanceDelay=2.5
DefaultCameraBlendTime=0.5
bAutoFocusSpeaker=False
```

{% hint style="info" %}
`TSoftObjectPtr`- und `TSoftClassPtr`-Referenzen werden beim ersten Dialog-Start lazy geladen. Es entsteht eine einmalige kurze Lade-Pause — plant das für deinen ersten Dialogstart ein.
{% endhint %}

## Siehe auch

- [Editor-Einstellungen](editor-settings.md) — Editor-spezifische Settings (Node-Farben, Debug-Highlights).
- [UI → UMG-Architektur](../ui/umg-architecture.md) — wie die UMG-Defaults das Widget-Layout steuern.
- [Audio → Babel-System](../audio/babel-system.md) — Babel-Details und Profil-Konfiguration.
