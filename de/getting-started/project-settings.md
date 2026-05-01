---
description: Alle Einstellungen unter Edit → Project Settings → Plugins → MayDialogue erklärt.
---

# Projekt-Einstellungen

MayDialogue registriert zwei Einstellungsabschnitte:

* **MayDialogue**: Runtime-Konfiguration. Wird in `DefaultGame.ini` gespeichert und ist im Shipping-Build eingefroren. Hier stellst du alles ein, was das laufende Spiel betrifft.
* **MayDialogue Editor**: Editor-Konfiguration. Wird in `EditorPerProjectUserSettings.ini` gespeichert und gilt nur im Editor. Hier stellst du Node-Farben und Editor-Verhalten ein.

Öffne beide unter **Edit → Project Settings → Plugins**.

> 📸 **Bild-Platzhalter:** `settings-overview.png`: Project Settings mit den beiden MayDialogue-Abschnitten in der Seitenleiste.
> *Setup:* Project Settings-Fenster. In der linken Kategorie-Leiste unter "Plugins" sind zwei Einträge sichtbar: "MayDialogue" und "MayDialogue Editor". Roter Pfeil auf beide Einträge.

---

## Runtime-Einstellungen: MayDialogue

### UI

| Property | Typ | Default | Wann ändern |
| --- | --- | --- | --- |
| `DefaultDialogueWidgetClass` | `TSoftClassPtr<UMayDialogueWidget>` | *(keins)* | Wenn du dein eigenes UMG-Widget verwenden willst. Leer lassen zeigt das Slate-Debug-Widget. |
| `bUseSlateDialogueWidget` | bool | `true` | Auf `false` setzen, sobald du `DefaultDialogueWidgetClass` gesetzt hast und das Debug-Widget nicht mehr brauchst. |
| `PanelBlurStrength` | float | `10.0` | Blur-Stärke des Slate-Debug-Panels. Nur relevant, solange du das Debug-Widget nutzt. |

{% hint style="info" %}
**Schnellste Wahl für den Start:** Lass `DefaultDialogueWidgetClass` leer und `bUseSlateDialogueWidget = true`. Das Debug-Widget erscheint automatisch, kein Widget-Blueprint nötig. Tausche es gegen dein eigenes aus, sobald das Design steht.
{% endhint %}

### UI: Komponenten-Defaults

Wenn du `DefaultDialogueWidgetClass` nutzt, aber nicht alle Sub-Widgets als `BindWidget` darin hardcodieren willst, kannst du Fallback-Klassen für jeden Slot setzen:

| Property | Zweck |
| --- | --- |
| `DefaultDialogFrameClass` | Haupt-Container (Hintergrund, Position) |
| `DefaultSpeakerWidgetClass` | Portrait + Sprecher-Name |
| `DefaultTextWidgetClass` | Typewriter-Textfeld |
| `DefaultChoiceButtonClass` | Einzelner Choice-Button |
| `DefaultChoiceListClass` | Container für alle Choice-Buttons |
| `DefaultSkipButtonClass` | "Weiter / Skip"-Button |

Details zur Architektur: [UI → UMG-Architektur](../ui/umg-architecture.md).

### Dialog-Defaults

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `DefaultAdvanceMode` | Enum | `Manual` | Wie SayLines standardmäßig weiterschalten. `Manual` = Spieler-Klick, `Timer` = nach N Sekunden, `AfterVoice` = wenn Audio endet, `Immediate` = sofort. |
| `DefaultAutoAdvanceDelay` | float | `3.0` | Wartezeit in Sekunden, wenn `DefaultAdvanceMode = Timer`. |

{% hint style="info" %}
`DefaultAdvanceMode` ist der globale Fallback. Jeder SayLine-Node kann diesen Wert per `AdvanceModeOverride` überschreiben.
{% endhint %}

### Typewriter

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `bEnableTypewriterEffect` | bool | `true` | Globaler Schalter. `false` = alle Texte erscheinen sofort. |
| `TypewriterCharsPerSecond` | float | `30.0` | Zeichen pro Sekunde. Kann per `<speed>` Rich-Text-Tag im Text überschrieben werden. |

### Input

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `bAllowSkipTypewriter` | bool | `true` | Spieler-Input zeigt den ganzen Text sofort. |
| `bAllowSkipVoiceLine` | bool | `true` | Spieler-Input bricht die laufende Voice-Wiedergabe ab und geht zum nächsten Node. |
| `bSwitchToUIInputDuringDialogue` | bool | `true` | Setzt Input-Mode auf `Game+UI`, damit Cursor und UI-Interaktion funktionieren. |
| `bShowMouseCursorDuringDialogue` | bool | `true` | Zeigt den Cursor für Choice-Buttons. Nur aktiv, wenn `bSwitchToUIInputDuringDialogue = true`. |

{% hint style="info" %}
**Hinweis für fortgeschrittene Setups mit eigenen Input-Modi:** Der Input-Mode wird nach Dialog-Ende hart auf `GameOnly` zurückgesetzt. Wenn dein Spiel vor dem Dialog einen anderen Modus nutzt (`UIOnly`, `GameAndUI`), musst du den Restore nach `OnDialogueEnded` manuell setzen. Für die meisten Projekte ist das Standardverhalten korrekt. Siehe [Known Issues](../troubleshooting/known-issues.md).
{% endhint %}

### Audio

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `DefaultSoundClass` | `TSoftObjectPtr<USoundClass>` | *(keins)* | SoundClass für alle Dialog-Voices, wenn kein Sprecher-Override greift. Zeige hier auf deine Projekt-eigene "Voice"-SoundClass. |
| `DefaultAttenuation` | `TSoftObjectPtr<USoundAttenuation>` | *(keins)* | 3D-Attenuation für Voices. Zeige hier auf dein Basis-Attenuation-Asset. |
| `bForce2D` | bool | `false` | Alle Dialog-Voices in 2D abspielen (Visual-Novel-Modus). |

### Babel Voice

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `bEnableBabelVoice` | bool | `false` | Aktiviert prozedurale Platzhalter-Stimmen für SayLines ohne Voice-Asset. |
| `DefaultBabelProfile` | `TSoftObjectPtr<UMayDialogueBabelProfile>` | *(Plugin-Default)* | Fallback-Profil für Sprecher ohne eigenes Babel-Profil. |

Details: [Audio → Babel-System](../audio/babel-system.md).

### Kamera

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `bAutoFocusSpeaker` | bool | `false` | Kamera schwenkt automatisch auf den aktuellen Sprecher, ohne dass du einen CameraFocus-Node setzen musst. |
| `DefaultCameraBlendTime` | float | `0.5` | Blend-Dauer in Sekunden, wenn ein CameraFocus-Node keine eigene Zeit angibt. |

### GAS: Lifecycle Cue Bindings

| Property | Typ | Bedeutung |
| --- | --- | --- |
| `LifecycleCueBindings` | `TArray<FMayDialogueCueLifecycleBinding>` | Deklarative Tabelle: wenn Dialog-Event X eintritt, feuere Gameplay Cue Y auf dem Instigator-ASC. |

Jeder Eintrag (`FMayDialogueCueLifecycleBinding`) hat:

| Feld | Bedeutung |
| --- | --- |
| `LifecycleEvent` | Wann der Cue feuert: `DialogueStarted`, `DialogueEnded`, `NodeReached`, `ChoiceMade`, `DialogueEventFired` |
| `EventTagFilter` | Nur bei `DialogueEventFired`: feuert nur, wenn das Event-Tag matched. Leer = immer. |
| `CueTag` | Der GameplayCue-Tag (muss unter `GameplayCue`-Hierarchie liegen) |
| `Mode` | `Execute` (One-Shot), `Add` (persistent), `Remove` |

Damit kannst du zum Beispiel einen Blur-Cue feuern, wenn ein Dialog startet, ohne Blueprint-Code schreiben zu müssen.

> 📸 **Bild-Platzhalter:** `settings-runtime-panel.png`: Project Settings mit dem ausgefüllten MayDialogue-Panel.
> *Setup:* Project Settings → MayDialogue. Sichtbar: Abschnitte "UI", "Dialogue Defaults", "Typewriter", "Input", "Audio", "Babel Voice", "Camera", "GAS Lifecycle Cues". Wichtigste Felder ausgefüllt: `DefaultSoundClass` und `DefaultAttenuation` mit Projekt-Assets, `bEnableTypewriterEffect = true`, `DefaultAdvanceMode = Manual`.

---

## Editor-Einstellungen: MayDialogue Editor

### Node-Farben

Hier kannst du die Default-Farben jedes Node-Typs im Graph-Editor anpassen. Die Einstellung ist pro-Projekt und wird in `EditorPerProjectUserSettings.ini` gespeichert.

Verfügbare Properties: `SayLineColor`, `PlayerChoiceColor`, `BranchColor`, `RandomLineColor`, `WaitColor`, `LinkColor`, `SubGraphColor`, `CameraFocusColor`, `AnimationColor`, `VariableColor`, `LogicColor`, `EntryExitColor`, `ErrorColor`.

{% hint style="info" %}
Sprecher-Farben werden **pro Dialog-Asset** im Speakers-Panel definiert und überschreiben `SayLineColor` für die Title-Bar der SayLine-Nodes dieses Sprechers. `SayLineColor` gilt nur für SayLines ohne konfigurierten Sprecher.
{% endhint %}

### Debug-Highlights

| Property | Bedeutung |
| --- | --- |
| `ActiveDebugColor` | Highlight-Farbe für den Node, der gerade im PIE-Debugger pausiert ist. |
| `HistoryDebugColor` | Highlight-Farbe für alle Nodes, die in dieser Debug-Session bereits besucht wurden. |

### Editor-Verhalten

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `bAutoCompileOnSave` | bool | `true` | Beim Speichern eines Dialog-Assets wird automatisch ein Compile-Pass ausgeführt. Fehler erscheinen sofort im Compiler Results Panel. |
| `bShowMinimapByDefault` | bool | `false` | Reserviert für eine zukünftige Minimap-Ansicht. |

---

## Empfohlene Mindestkonfiguration für ein neues Projekt

Wenn du ein Projekt von Null aufbaust, reichen zunächst vier Einstellungen:

1. **`DefaultSoundClass`** auf deine Projekt-eigene Voice-SoundClass zeigen lassen.
2. **`DefaultAttenuation`** auf dein Basis-Attenuation-Asset zeigen lassen.
3. **`DefaultSpeakerWidgetClass`** und **`DefaultDialogFrameClass`** auf deine Blueprint-Subklassen zeigen lassen, sobald du erste UI-Widgets gebaut hast.
4. Alles andere kann bei den Defaults bleiben, bis du gezielt etwas anpassen musst.

> 📸 **Bild-Platzhalter:** `settings-minimum-config.png`: Project Settings mit der Mindestkonfiguration für ein neues Projekt.
> *Setup:* MayDialogue-Settings-Panel. Nur `DefaultSoundClass` und `DefaultAttenuation` sind mit Assets belegt. Alle anderen Felder leer oder auf Default-Wert. Roter Pfeil auf `DefaultSoundClass` und `DefaultAttenuation`.
