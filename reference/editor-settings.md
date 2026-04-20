# Editor-Einstellungen (Referenz)

Kompakt-Referenz von `UMayDialogueEditorSettings`. Nur im Editor-Build relevant (Modul `MayDialogueEditor`, `UncookedOnly`).

* **Klasse**: `UMayDialogueEditorSettings` (`UDeveloperSettings`)
* **Config-Category**: `EditorPerProjectUserSettings`
* **Config-File**: `EditorPerProjectUserSettings.ini`
* **Section**: `/Script/MayDialogueEditor.MayDialogueEditorSettings`
* **UI-Pfad im Editor**: *Edit → Editor Preferences → Plugins → MayDialogue Editor*

Diese Settings sind **pro User**, nicht pro Projekt – sie werden nicht mit-committet. Wenn dein Team einheitliche Node-Farben haben soll, ziehe sie nach `DefaultEditorPerProjectUserSettings.ini` und committe die.

## Node-Farben

Alle Werte sind `FLinearColor`, angezeigt als Color-Picker. Der jeweils benannte Node-Typ bekommt die Title-Bar in dieser Farbe.

| Property | Node(s) | Default |
| --- | --- | --- |
| `SayLineColor` | SayLine | helles Blau |
| `PlayerChoiceColor` | PlayerChoice | Orange |
| `BranchColor` | Branch | Grün |
| `RandomLineColor` | RandomLine | Lila |
| `WaitColor` | Wait | Grau |
| `LinkColor` | Link | Türkis |
| `SubGraphColor` | SubGraph | dunkles Türkis |
| `CameraFocusColor` | CameraFocus, CameraShake | Magenta |
| `AnimationColor` | PlayAnimation | Gelb |
| `VariableColor` | SetVariable | Senf |
| `LogicColor` | Branch (Logik-Fallback) / FireEvent | Hell-Grün |
| `EntryExitColor` | Entry, Exit | Weiß |
| `ErrorColor` | Fehler-Tint (Node mit Compile-Error) | Rot |

{% hint style="info" %}
**Reset-Button**: Neben jeder Farbe im Editor-Settings-Panel ist ein Pfeil-Icon, das auf den CDO-Default zurücksetzt – praktisch, wenn du dich verklickt hast.
{% endhint %}

## Debug-Highlights

Nur während einer PIE-Session im [Debugger](../editor/debugger.md) sichtbar.

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `ActiveDebugColor` | `FLinearColor` | kräftiges Gelb | Highlight-Rahmen um den Node, bei dem die Instance gerade pausiert. |
| `HistoryDebugColor` | `FLinearColor` | blasses Gelb | Rahmen um alle während dieser Debug-Session besuchten Nodes. |

## Editor-Verhalten

| Property | Typ | Default | Bedeutung |
| --- | --- | --- | --- |
| `bAutoCompileOnSave` | `bool` | `true` | Speichern eines Dialog-Assets triggert automatisch eine Compile-Pass. |
| `bShowMinimapByDefault` | `bool` | `false` | *(reserviert – geplantes Feature, derzeit no-op.)* |

## Programmatischer Zugriff

```cpp
#if WITH_EDITOR
const UMayDialogueEditorSettings* ES = GetDefault<UMayDialogueEditorSettings>();
const FLinearColor ChoiceColor = ES->PlayerChoiceColor;
#endif
```

Blueprint-Zugriff existiert nicht – die Klasse ist `EditorOnly`.

## Einheitliche Team-Settings erzwingen

1. Dein persönliches `EditorPerProjectUserSettings.ini` bearbeiten, bis alles so aussieht, wie ihr es haben wollt.
2. Den relevanten Block extrahieren:
   ```ini
   [/Script/MayDialogueEditor.MayDialogueEditorSettings]
   SayLineColor=(R=0.2,G=0.55,B=0.9,A=1)
   PlayerChoiceColor=(R=0.95,G=0.55,B=0.1,A=1)
   bAutoCompileOnSave=True
   ```
3. Diese Zeilen in `Config/DefaultEditorPerProjectUserSettings.ini` committen.

Damit bekommen Team-Kolleg*innen beim Repo-Pull die gleichen Farben, können sie aber lokal überschreiben.

## Reset nach Plugin-Update

Nach einem Plugin-Update, bei dem sich Default-Farben geändert haben, kannst du den Reset pro Property ausführen (Pfeil-Icon) oder pro Config-Block:

```ini
# Alle MayDialogueEditor-Defaults wiederherstellen – einfach löschen:
[/Script/MayDialogueEditor.MayDialogueEditorSettings]
```

Editor neu starten, der CDO schreibt die Defaults beim ersten Zugriff zurück.

## Roadmap / Bekannte Lücken

* `bShowMinimapByDefault` ist **reserviert**; die Minimap ist im Asset-Editor aktuell ein toggle-Button ohne persistente Setting-Bindung.
* Node-Farben werden aktuell **pro Node-Typ** gesetzt. Pro-Instance-Farben (individuelle Nodes umfärben) sind ein Backlog-Item.
* Node-Title-Text-Farbe ist nicht konfigurierbar – sie passt sich automatisch dem Helligkeits-Kontrast der gewählten Title-Bar-Farbe an.

## Siehe auch

* [Projekt-Einstellungen](project-settings.md) – alle Runtime-Settings.
* [Editor → Komfort-Features](../editor/comfort-features.md) – Shortcuts und UX-Details des Asset-Editors.
* [Editor → Debugger](../editor/debugger.md) – wo die Debug-Farben sichtbar werden.
