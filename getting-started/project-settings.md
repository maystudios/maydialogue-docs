# Projekt-Einstellungen

MayDialogue liefert zwei `UDeveloperSettings`-Abschnitte: eine **Runtime-Konfiguration** (im Shipping-Build eingefroren) und eine **Editor-Konfiguration** (nur Editor-only). Beide findest du unter **Edit → Project Settings**.

## Runtime: „Plugins → MayDialogue"

Quelle: [`UMayDialogueSettings`](../reference/project-settings.md). Konfigurationsdatei: `DefaultGame.ini` (Kategorie `Game`).

### Widget

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `DefaultDialogueWidgetClass` | `TSoftClassPtr<UMayDialogueWidget>` | *(keins)* | UMG-Widget, das beim Dialog-Start automatisch aus dem Viewport angezeigt wird. Leer → Fallback auf Slate-Debug-Widget. |
| `bUseSlateDialogueWidget` | bool | `true` | Aktiviert das Slate-Debug-Widget, solange kein UMG gesetzt ist. |
| `PanelBlurStrength` | float | `4.0` | Blur-Strength der Slate-Debug-Panel-Flächen. |

### UMG-Komponenten-Defaults

Die komponenten-basierte UMG-Architektur sucht einzelne Widget-Klassen, wenn `DefaultDialogueWidgetClass` keine BindWidget-Slots setzt.

| Property | Typ |
| --- | --- |
| `DefaultDialogFrameClass` | `TSoftClassPtr<UMayDialogueWidget_DialogFrame>` |
| `DefaultSpeakerWidgetClass` | `TSoftClassPtr<UMayDialogueWidget_Speaker>` |
| `DefaultTextWidgetClass` | `TSoftClassPtr<UMayDialogueWidget_Text>` |
| `DefaultChoiceButtonClass` | `TSoftClassPtr<UMayDialogueWidget_ChoiceButton>` |
| `DefaultChoiceListClass` | `TSoftClassPtr<UMayDialogueWidget_ChoiceList>` |
| `DefaultSkipButtonClass` | `TSoftClassPtr<UMayDialogueWidget_SkipButton>` |

Details siehe [UI → UMG-Architektur](../ui/umg-architecture.md).

### Dialog-Defaults

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `DefaultAdvanceMode` | `EMayDialogueAdvanceMode` | `Manual` | Advance-Mode für SayLines ohne eigenen Override. |
| `DefaultAutoAdvanceDelay` | float | `3.0` | Timer-Delay (Sekunden) für AdvanceMode `Timer`. |

### Typewriter

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `bEnableTypewriterEffect` | bool | `true` | Globaler Typewriter-Schalter. |
| `TypewriterCharsPerSecond` | float | `30.0` | Zeichen pro Sekunde, wenn kein `<speed>`-Tag greift. |

### Input

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `bAllowSkipTypewriter` | bool | `true` | Spieler-Input skippt den Typewriter sofort auf das Text-Ende. |
| `bAllowSkipVoiceLine` | bool | `false` | Spieler-Input bricht die laufende Voice-Wiedergabe ab und geht zum nächsten Node. |
| `bSwitchToUIInputDuringDialogue` | bool | `true` | Setzt Input-Mode auf `GameAndUI` für die Dialog-Dauer. |
| `bShowMouseCursorDuringDialogue` | bool | `true` | Zeigt den Cursor für Choice-Buttons. |

{% hint style="warning" %}
**Input-Mode-Restore ist derzeit hartkodiert auf `GameOnly`.** Wenn dein Spiel andere Modi (z.B. `UIOnly`) benutzt, musst du den Restore nach Dialog-Ende manuell nachziehen. Details im [Backlog-Item 4](../troubleshooting/known-issues.md).
{% endhint %}

### Audio

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `DefaultSoundClass` | `TSoftObjectPtr<USoundClass>` | *(keins)* | Sound-Class für alle Dialog-Voices, falls kein Speaker-Override greift. |
| `DefaultAttenuation` | `TSoftObjectPtr<USoundAttenuation>` | *(keins)* | 3D-Attenuation für 3D-Voices. |
| `bForce2D` | bool | `false` | Globaler Schalter: alle Dialog-Voices in 2D wiedergeben (Visual-Novel-Modus). |

### Babel-Voice

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `bEnableBabelVoice` | bool | `true` | Aktiviert Babel-Synthese für Lines ohne Voice-Asset. |
| `DefaultBabelProfile` | `TSoftObjectPtr<UMayDialogueBabelProfile>` | *(Plugin-Default)* | Fallback-Profil für Sprecher ohne eigenes Profil. |

Details siehe [Audio → Babel-System](../audio/babel-system.md).

### Kamera

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `bAutoFocusSpeaker` | bool | `false` | Automatischer CameraFocus auf den aktuellen SayLine-Speaker, ohne expliziten CameraFocus-Node. |
| `DefaultCameraBlendTime` | float | `0.75` | Blend-Dauer, wenn ein CameraFocus-Node keine eigene angibt. |

## Editor: „Plugins → MayDialogue Editor"

Quelle: [`UMayDialogueEditorSettings`](../reference/editor-settings.md). Konfigurationsdatei: `EditorPerProjectUserSettings.ini`.

### Node-Farben

Pro Node-Typ kann die Title-Bar-Farbe angepasst werden (übersteuert den Default aus dem Node-Katalog):

`SayLineColor`, `PlayerChoiceColor`, `BranchColor`, `RandomLineColor`, `WaitColor`, `LinkColor`, `SubGraphColor`, `CameraFocusColor`, `AnimationColor`, `VariableColor`, `LogicColor`, `EntryExitColor`, `ErrorColor`.

### Debug-Highlights

| Property | Bedeutung |
| --- | --- |
| `ActiveDebugColor` | Highlight-Farbe für den aktuell pausierten Node im PIE-Debugger. |
| `HistoryDebugColor` | Highlight-Farbe für während der Debug-Session besuchte Nodes. |

### Editor-Verhalten

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `bAutoCompileOnSave` | bool | `true` | Speichern eines Dialog-Assets triggert automatisch eine Compile-Pass. |
| `bShowMinimapByDefault` | bool | `false` | *(reserviert, siehe Roadmap)* |

## Empfohlene Minimum-Konfiguration

Ein neues Projekt braucht für den produktiven Einsatz meistens nur:

1. **Speaker-Widget** als eigene Blueprint-Subklasse anlegen (Portrait + Name-Style nach Design).
2. `DefaultSpeakerWidgetClass` auf die Subklasse zeigen.
3. **DialogFrame-Widget** als Blueprint-Subklasse anlegen (Box-Position, Background-Art-Work).
4. `DefaultDialogFrameClass` auf die Subklasse zeigen.
5. **DefaultSoundClass** und **DefaultAttenuation** auf Projekt-eigene Assets zeigen, damit Dialog-Voice im gleichen Mix landet wie andere Sounds.

Alles andere kann erst mal bei den Defaults bleiben.
