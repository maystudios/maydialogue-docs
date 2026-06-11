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
> *Setup:* Editor geöffnet, Edit → Project Settings → Plugins → MayDialogue. Vollständiger Screenshot des Settings-Panels mit allen Kategorien sichtbar: Widget, UMG-Komponenten-Defaults, Dialog-Defaults, Typewriter, Input, Enhanced Input, Audio, Babel-Voice, Kamera, Wait, Barrierefreiheit, GAS Lifecycle Cues. Roter Pfeil zeigt auf den Navigations-Pfad oben im Settings-Fenster.

---

## Widget

| Property | Typ | Default | Bedeutung |
|---|---|---|---|
| `DefaultDialogueWidgetClass` | `TSoftClassPtr<UMayDialogueWidget>` | *(leer)* | UMG-Widget das als Dialog-Overlay angezeigt wird. Leer → Slate-Debug-Fallback. |
| `bUseSlateDialogueWidget` | `bool` | `true` | `true` → das eingebaute Slate-Widget spawnen; `false` → stattdessen das UMG-Widget aus `DefaultDialogueWidgetClass`. |
| `PanelBlurStrength` | `float` | `10.0` | Blur-Stärke der Slate-Panels (nur wirksam wenn `bUseSlateDialogueWidget` true ist). |

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
| `bAllowSkipVoiceLine` | `bool` | `true` | Spieler-Input stoppt laufende Voice und advanced weiter. |
| `bSwitchToUIInputDuringDialogue` | `bool` | `true` | Input-Mode `GameAndUI` für die Dialog-Dauer. |
| `bShowMouseCursorDuringDialogue` | `bool` | `true` | Maus-Cursor während des Dialogs einblenden (nur wenn `bSwitchToUIInputDuringDialogue`). |

---

## Enhanced Input

Optionale Enhanced-Input-Integration. Jedes Feld außer `bUseEnhancedInput` graut aus (`EditCondition`), wenn der Master-Switch aus ist. Die hardcoded Tasten bleiben als Fallback, sodass Dialog-Input auch ohne Konfiguration funktioniert. Siehe [Runtime → Enhanced Input](../runtime/enhanced-input.md) für das vollständige Schritt-für-Schritt-Setup.

| Property | Typ | Default | Bedeutung |
|---|---|---|---|
| `bUseEnhancedInput` | `bool` | `true` | Master-Switch. Aus = nur die Fallback-Tasten gelten. |
| `DialogueMappingContext` | `TSoftObjectPtr<UInputMappingContext>` | *(leer)* | IMC, das beim Dialog-Start hinzugefügt und am Ende entfernt wird. |
| `MappingContextPriority` | `int32` | `100` | Priorität, mit der das IMC gepusht wird (höher schlägt niedrigere Kontexte). |
| `AdvanceAction` | `TSoftObjectPtr<UInputAction>` | *(leer)* | Advance / Typewriter-Skip. |
| `SkipAction` | `TSoftObjectPtr<UInputAction>` | *(leer)* | Voice- / Line-Skip. |
| `AbortAction` | `TSoftObjectPtr<UInputAction>` | *(leer)* | Dialog abbrechen. |
| `ChoiceUpAction` | `TSoftObjectPtr<UInputAction>` | *(leer)* | Choice-Cursor hoch. |
| `ChoiceDownAction` | `TSoftObjectPtr<UInputAction>` | *(leer)* | Choice-Cursor runter. |
| `ChoiceConfirmAction` | `TSoftObjectPtr<UInputAction>` | *(leer)* | Hervorgehobene Choice bestätigen. |
| `ChoiceByIndexActions` | `TArray<TSoftObjectPtr<UInputAction>>` | *(leer)* | Direkt-Auswahl: Index 0 = Choice 1, … (bis zu neun). |
| `bAutoDetectInputDevice` | `bool` | `true` | Tastatur/Gamepad automatisch erkennen und Skip-Glyph + Slate-Hint speisen. |

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
| `DefaultCameraBlendTime` | `float` | `0.5` | Blend-Dauer in Sekunden für CameraFocus ohne explizite Angabe. |
| `DefaultAnchorRestoreBlendTime` | `float` | `0.5` | Blend-Zeit beim Wiederherstellen des ursprünglichen ViewTargets nach einem Camera-Anchor-/Shot-Focus bei Dialog-Ende (0 = snap). |

---

## Wait

| Property | Typ | Default | Bedeutung |
|---|---|---|---|
| `LatchedEventWindowSeconds` | `float` | `0.1` | Wie weit zurück (Sekunden) ein Gameplay-Event gefeuert worden sein darf, bevor ein Wait-Node aktiviert wird, und trotzdem als "gerade gefeuert" zählt (Latched-Event-Catch-up). Clamped `0.0–5.0`. |

---

## Barrierefreiheit (1.0)

| Property | Typ | Default | Bedeutung |
|---|---|---|---|
| `DialogueTextScale` | `float` | `1.0` | Einheitlicher Text-Skalierungsfaktor für alle Dialog-Schriftgrößen (Nachrichtentext, Speaker-Name, Choice-Buttons) auf Slate- und UMG-Widget-Schicht. Begrenzt auf `0,5–3,0`. |
| `bColorblindSafeChoiceCues` | `bool` | `true` | Wenn true, zeigen nicht verfügbare (gesperrte) Choices ein Schloss-Glyph-Präfix, damit sie ohne Farbabhängigkeit von verfügbaren Choices unterscheidbar sind. |

`DialogueTextScale` wird von `MayDialogue::Accessibility::ClampTextScale(S->DialogueTextScale)` im eingebauten Slate-Widget (`SMayDialogueWidget`) und in den Standard-UMG-Komponenten-Widgets ausgelesen. Es muss kein Widget ersetzt werden, um größeren Text zu verwenden — den Scale hier setzen und er gilt projekt-weit.

`bColorblindSafeChoiceCues` setzt einem Unicode-Schloss-Glyph (🔒) vor den Text nicht verfügbarer Choice-Buttons via `MayDialogue::Accessibility::GetColorblindChoicePrefix(...)`. Projekte mit eigenem farbunabhängigem Verfügbarkeits-Indikator können dies deaktivieren.

**Screen-Reader-Unterstützung:** Beide Widget-Schichten rufen `SetAccessibleBehavior(EAccessibleBehavior::Custom, ...)` für jedes angezeigte Element auf, unter Verwendung von `MayDialogue::Accessibility::MakeSpeakerAccessibleText`, `MakeDialogueTextAccessibleText` und `MakeChoiceAccessibleText`. Der Accessible-Text für nicht verfügbare Choices enthält die Phrase "Gesperrte Wahl" und, wenn ein Grund angegeben wird, die Anforderungsbeschreibung — so gibt assistive Technologie Kontext aus, auch ohne das visuelle Schloss-Glyph.

```ini
# DefaultGame.ini-Beispiel
[/Script/MayDialogue.MayDialogueSettings]
DialogueTextScale=1.5
bColorblindSafeChoiceCues=True
```

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
DialogueTextScale=1.0
bColorblindSafeChoiceCues=True
```

{% hint style="info" %}
`TSoftObjectPtr`- und `TSoftClassPtr`-Referenzen werden beim ersten Dialog-Start lazy geladen. Es entsteht eine einmalige kurze Lade-Pause — plant das für deinen ersten Dialogstart ein.
{% endhint %}

---

## Native GameplayTags im Plugin (1.0)

Das Plugin registriert beim Modul-Start einen minimalen Satz nativer `GameplayTag`-Werte, damit die Editor-Tag-Picker befüllt sind und die Schnellstart-Anleitung ohne manuelle Tag-Tabellen-Pflege funktioniert.

### Speaker-Tags

| C++-Name | Tag-String | Zweck |
|---|---|---|
| `TAG_MayDialogue_Speaker_Guard` | `Dialogue.Speaker.Guard` | Beispiel-Speaker-Tag — entspricht dem mitgelieferten `DA_Greeting_Simple`-Sample. |
| `TAG_MayDialogue_Speaker_Player` | `Dialogue.Speaker.Player` | Beispiel-Tag für den Spieler-Charakter. |
| `TAG_MayDialogue_Speaker_NPC` | `Dialogue.Speaker.NPC` | Generischer Beispiel-Tag für unbenannte NPCs. |

### Lifecycle-GameplayCue-Tags

| C++-Name | Tag-String | Entspricht |
|---|---|---|
| `TAG_GameplayCue_Dialogue` | `GameplayCue.Dialogue` | Root-Tag für alle MayDialogue-Lifecycle-Cues. |
| `TAG_GameplayCue_Dialogue_Started` | `GameplayCue.Dialogue.Started` | `EMayDialogueLifecycleEvent::DialogueStarted` |
| `TAG_GameplayCue_Dialogue_Ended` | `GameplayCue.Dialogue.Ended` | `EMayDialogueLifecycleEvent::DialogueEnded` |
| `TAG_GameplayCue_Dialogue_NodeReached` | `GameplayCue.Dialogue.NodeReached` | `EMayDialogueLifecycleEvent::NodeReached` |
| `TAG_GameplayCue_Dialogue_ChoiceMade` | `GameplayCue.Dialogue.ChoiceMade` | `EMayDialogueLifecycleEvent::ChoiceMade` |
| `TAG_GameplayCue_Dialogue_EventFired` | `GameplayCue.Dialogue.EventFired` | `EMayDialogueLifecycleEvent::DialogueEventFired` |

Tags werden in `MayDialogueGameplayTags.h` deklariert (einbinden, um per extern-Variable-Namen auf sie zuzugreifen). Definitionen liegen in `MayDialogueGameplayTags.cpp` und werden automatisch registriert.

**Projektspezifische Tags hinzufügen:** Eigene Tags in der `DefaultGameplayTags.ini` des Projekts oder über *Project Settings → GameplayTags* neben den Plugin-Tags anlegen. Die Plugin-Tags sind bewusst minimal und klar unter `Dialogue.*` geschützt, um keine Projekttag-Bäume zu beeinflussen.

**Lifecycle-Cue-Bindungen:** Die `GameplayCue.Dialogue.*`-Tags mit eigenen GAS-gesteuerten Cues in *Project Settings → MayDialogue → GAS → Lifecycle Cue Bindings* verbinden. Siehe [GAS-Integration](../gas/README.md) für die vollständige Bindungs-Einrichtung.

---

## Siehe auch

- [Editor-Einstellungen](editor-settings.md) — Editor-spezifische Settings (Node-Farben, Debug-Highlights).
- [UI → UMG-Architektur](../ui/umg-architecture.md) — wie die UMG-Defaults das Widget-Layout steuern.
- [Audio → Babel-System](../audio/babel-system.md) — Babel-Details und Profil-Konfiguration.
- [GAS-Integration](../gas/README.md) — Lifecycle-Cue-Bindungen und GAS-Requirements.
