---
description: Alle Felder von UMayDialogueEditorSettings — Node-Farben, Debug-Highlights, Editor-Verhalten.
---

# Editor-Einstellungen (Referenz)

Kompakt-Referenz von `UMayDialogueEditorSettings`. Nur im Editor-Build relevant.

- **Klasse**: `UMayDialogueEditorSettings` (`UDeveloperSettings`)
- **Config-Datei**: `EditorPerProjectUserSettings.ini`
- **Section**: `/Script/MayDialogueEditor.MayDialogueEditorSettings`
- **UI-Pfad**: *Edit → Editor Preferences → Plugins → MayDialogue Editor*

{% hint style="info" %}
Diese Settings sind **pro User**, nicht pro Projekt — sie werden nicht mit dem Repository mitgeliefert. Wenn dein Team einheitliche Node-Farben haben soll, committe die Werte nach `Config/DefaultEditorPerProjectUserSettings.ini`.
{% endhint %}

> 📸 **Bild-Platzhalter:** `editor-settings-panel.png` — Screenshot des Editor-Preferences-Panels.
> *Setup:* Editor geöffnet, Edit → Editor Preferences → Plugins → MayDialogue Editor. Vollständiger Screenshot mit Color-Picker-Feldern für alle Node-Typen sichtbar. Roter Pfeil zeigt auf den Navigations-Pfad und einen einzelnen Color-Picker-Button zur Illustration.

---

## Node-Farben

Alle Werte sind `FLinearColor`, angezeigt als Color-Picker im Panel. Die jeweilige Node-Typ-Gruppe bekommt die Title-Bar in dieser Farbe.

| Property | Gilt für Node(s) | Default |
|---|---|---|
| `SayLineColor` | SayLine | Helles Blau |
| `PlayerChoiceColor` | PlayerChoice | Orange |
| `BranchColor` | Branch | Grün |
| `RandomLineColor` | RandomLine | Lila |
| `WaitColor` | Wait | Grau |
| `LinkColor` | Link | Türkis |
| `SubGraphColor` | SubGraph | Dunkles Türkis |
| `CameraFocusColor` | CameraFocus, CameraShake | Magenta |
| `AnimationColor` | PlayAnimation | Gelb |
| `VariableColor` | SetVariable | Senf |
| `LogicColor` | Branch (Logik-Fallback), FireEvent | Hell-Grün |
| `EntryExitColor` | Entry, Exit | Weiß |
| `ErrorColor` | Nodes mit Compile-Fehler (Fehler-Tint) | Rot |

> 📸 **Bild-Platzhalter:** `node-colors-overview.png` — Dialog-Graph mit einem Node jedes Typs nebeneinander.
> *Setup:* Einen Dialog-Asset öffnen der mindestens enthält: Entry (weiß), SayLine (blau), PlayerChoice (orange), Branch (grün), FireEvent (hell-grün), Wait (grau), Exit (weiß). Alle Nodes kompakt angeordnet, keine Verbindungen nötig. Zeigt wie die Farben im echten Editor aussehen.

{% hint style="info" %}
Neben jedem Color-Picker im Editor-Settings-Panel ist ein Pfeil-Icon, das den Wert auf den CDO-Default zurücksetzt.
{% endhint %}

---

## Debug-Highlights

Sichtbar nur während einer PIE-Session im Debugger.

| Property | Typ | Default | Bedeutung |
|---|---|---|---|
| `ActiveDebugColor` | `FLinearColor` | Kräftiges Gelb | Highlight-Rahmen um den Node, bei dem die Instance gerade pausiert ist. |
| `HistoryDebugColor` | `FLinearColor` | Blasses Gelb | Rahmen um alle Nodes, die während dieser PIE-Session besucht wurden. |

> 📸 **Bild-Platzhalter:** `debug-highlights-pie.png` — Dialog-Graph während PIE mit aktiver Debug-Session.
> *Setup:* Dialog-Asset im Editor geöffnet, PIE läuft, Dialog aktiv. Ein Node ist mit kräftig gelbem Rahmen markiert (aktuelle Position). Zwei oder drei vorherige Nodes haben blasse Gelb-Rahmen (History). Screenshot zeigt den Graphen mit diesen Highlights.

---

## Editor-Verhalten

| Property | Typ | Default | Bedeutung |
|---|---|---|---|
| `bAutoCompileOnSave` | `bool` | `true` | Speichern eines Dialog-Assets triggert automatisch eine Compile-Pass. |
| `bShowMinimapByDefault` | `bool` | `false` | Reserviert — Minimap-Feature ist noch nicht aktiv. |

---

## Layout & Auto-Grid

Steuert das **Auto-Layout** (Toolbar → Auto-Layout) und das optionale **Auto-Grid**. Siehe auch [Komfort-Funktionen](../editor/comfort-features.md).

| Property | Typ | Default | Bedeutung |
|---|---|---|---|
| `AutoLayoutHorizontalSpacing` | `float` | `420` | Horizontaler Abstand (px) zwischen aufeinanderfolgenden Node-Spalten. |
| `AutoLayoutVerticalSpacing` | `float` | `180` | Mindest-Abstand (px, Mitte zu Mitte) zwischen Nodes in einer Spalte. |
| `AutoLayoutInterRowGap` | `float` | `80` | Zusätzlicher Abstand (px) über/unter jedem Node, oben auf die Knotenhöhe. |
| `AutoLayoutGridSize` | `float` | `16` | Raster (px), auf das Positionen nach dem Auto-Layout gesnappt werden (wenn Auto-Grid aus). |
| `AutoLayoutMaxBarycenterSweeps` | `int32` | `24` | Max. Anzahl Barycenter-Durchläufe zur Kreuzungs-Minimierung. |
| `bAutoLayoutInsertReroutes` | `bool` | `true` | Auto-Layout fügt entlang langer Kanten ausgerichtete Reroute-Knoten ein – **nur an echten Knickpunkten** (gerade Kanten bleiben unangetastet). Auto-Reroutes werden bei jedem Lauf neu aufgebaut; eigene Reroutes bleiben erhalten. Auch als Toolbar-Umschalter. |
| `bEnableAutoGrid` | `bool` | `false` | Aktiviert ein gröberes Ausrichtungsraster (größer als die Engine-Standard-16 px). An: Nodes snappen beim Ziehen, Einfügen, Erstellen und Auto-Layout an `AutoGridSize`, und das Hintergrundraster wird in dieser Größe gezeichnet. |
| `AutoGridSize` | `int32` | `64` | Rastergröße (px) bei aktivem Auto-Grid (16–512; ein Vielfaches von 16 wird empfohlen). |

> Auto-Layout nutzt das Brandes-Köpf-„Fast & Simple"-Verfahren: verbundene Nodes werden vertikal ausgerichtet, sodass Kanten möglichst gerade verlaufen, und die Knotenhöhe verhindert Überlappungen. Das Ergebnis ist deterministisch.

---

## Einheitliche Team-Settings

1. Lokal die Settings nach Wunsch konfigurieren.
2. Den Block aus deiner `EditorPerProjectUserSettings.ini` extrahieren:

```ini
[/Script/MayDialogueEditor.MayDialogueEditorSettings]
SayLineColor=(R=0.2,G=0.55,B=0.9,A=1)
PlayerChoiceColor=(R=0.95,G=0.55,B=0.1,A=1)
BranchColor=(R=0.2,G=0.7,B=0.2,A=1)
bAutoCompileOnSave=True
```

3. Diese Zeilen in `Config/DefaultEditorPerProjectUserSettings.ini` committen.

Team-Mitglieder bekommen beim nächsten Repo-Pull die gleichen Farben, können sie aber lokal überschreiben.

---

## Programmatischer Zugriff (nur C++/Editor)

```cpp
#if WITH_EDITOR
const UMayDialogueEditorSettings* ES = GetDefault<UMayDialogueEditorSettings>();
FLinearColor ChoiceColor = ES->PlayerChoiceColor;
#endif
```

Blueprint-Zugriff existiert nicht — die Klasse ist EditorOnly.

---

## Reset nach Plugin-Update

Falls sich Default-Farben nach einem Plugin-Update geändert haben — einfach den Settings-Block aus der `EditorPerProjectUserSettings.ini` löschen und den Editor neu starten. Die Defaults werden beim ersten Zugriff neu geschrieben.

## Siehe auch

- [Projekt-Einstellungen](project-settings.md) — alle Runtime-Settings.
- [Editor → Debugger](../editor/debugger.md) — wo die Debug-Farben im Graphen sichtbar werden.
