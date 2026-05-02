---
description: "Plugin in dein Projekt einbinden: Schritt für Schritt."
---

# Installation

MayDialogue ist ein UE-Plugin mit drei Modulen. Die Installation läuft in fünf Schritten und dauert rund fünf Minuten.

## Voraussetzungen

| Komponente | Mindestversion | Hinweis |
| --- | --- | --- |
| Unreal Engine | 5.7 | Binary- oder Source-Build |
| Visual Studio 2022 | 17.9+ | Nur wenn dein Projekt ein C++-Modul hat |
| JetBrains Rider | 2024.3+ | Optional |

**Plugin-Dependencies** (werden automatisch mitaktiviert, keine manuelle Aktivierung nötig):

* `GameplayAbilities`
* `GameplayTagsEditor`
* `StructUtils`
* `EnhancedInput`

## Schritt 1: Plugin-Ordner kopieren

1. Schließe den Unreal-Editor, falls er läuft.
2. Navigiere in deinem Projekt-Ordner zu `Plugins/`. Falls der Ordner nicht existiert, lege ihn an.
3. Kopiere den gesamten `MayDialogue`-Ordner (inklusive `MayDialogue.uplugin`) nach `Plugins/MayDialogue/`.

Deine Verzeichnisstruktur danach:

```
MeinProjekt/
├── MeinProjekt.uproject
├── Source/
├── Content/
└── Plugins/
    └── MayDialogue/
        ├── MayDialogue.uplugin
        ├── Source/
        ├── Content/
        └── Resources/
```

> 📸 **Bild-Platzhalter:** `installation-folder-structure.png`: Windows Explorer mit dem geöffneten `Plugins/`-Ordner des Projekts.
> *Setup:* Explorer-Fenster zeigt `Plugins/MayDialogue/` mit den Unterordnern `Source/`, `Content/`, `Resources/` und der Datei `MayDialogue.uplugin`. Roter Pfeil auf `MayDialogue.uplugin`.

## Schritt 2: Projekt-Dateien neu generieren

Nur notwendig, wenn dein Projekt ein C++-Modul hat (`.uproject` enthält `"Modules"` mit mindestens einem Eintrag).

{% tabs %}
{% tab title="Windows" %}
Rechtsklick auf `.uproject` → **Generate Visual Studio project files**, oder in einer Shell:

```bash
"C:/Program Files/Epic Games/UE_5.7/Engine/Build/BatchFiles/GenerateProjectFiles.bat" \
  -project="C:/Pfad/Zu/MeinProjekt.uproject" -game -rocket
```
{% endtab %}

{% tab title="macOS / Linux" %}
```bash
./UnrealEngine/GenerateProjectFiles.sh -project=/pfad/zu/MeinProjekt.uproject -game
```
{% endtab %}
{% endtabs %}

## Schritt 3: Projekt bauen

Öffne die `.sln` in Rider oder Visual Studio und baue das **Editor-Target** (`MeinProjektEditor`, Development, Win64).

Alternativ per Batch:

```bash
"C:/Program Files/Epic Games/UE_5.7/Engine/Build/BatchFiles/Build.bat" \
  MeinProjektEditor Win64 Development "-project=C:/Pfad/Zu/MeinProjekt.uproject"
```

{% hint style="info" %}
**Reine Blueprint-Projekte** können diesen Schritt überspringen. UE kompiliert beim ersten Editor-Start die nötigen Binaries automatisch.
{% endhint %}

## Schritt 4: Editor öffnen und Plugin prüfen

1. Öffne das Projekt.
2. **Edit → Plugins** aufrufen.
3. Kategorie **Gameplay** wählen. Der Eintrag **MayDialogue** muss sichtbar und aktiviert sein (Checkbox gesetzt).

> 📸 **Bild-Platzhalter:** `installation-plugin-dialog.png`: Der Plugins-Dialog mit dem MayDialogue-Eintrag.
> *Setup:* Plugins-Fenster geöffnet, Kategorie "Gameplay" ausgewählt. MayDialogue-Eintrag sichtbar mit aktivierter Checkbox, dem Beta-Banner darunter und dem Plugin-Beschreibungstext. Roter Pfeil auf die aktivierte Checkbox.

{% hint style="info" %}
**Beta-Flag.** MayDialogue ist als `IsBetaVersion: true` markiert. UE zeigt einen Warn-Banner im Plugin-Dialog. Du kannst ihn ignorieren.
{% endhint %}

## Schritt 5: Content-Pfad freigeben

MayDialogue liefert ein Content-Verzeichnis mit Default-Assets. Damit der Content-Browser es anzeigt:

1. **Content Browser** öffnen.
2. Kleines **Settings-Icon** rechts oben klicken.
3. **Show Plugin Content** aktivieren.

## Installation prüfen

Erstelle einen Ordner `Content/Dialogues/` und:

1. Rechtsklick im Content Browser → **May Dialogue → Dialogue Asset**.
2. Asset benennen: `DA_Test_Install`.
3. Doppelklick öffnet den Asset-Editor mit einem leeren Graph und einem **Entry**-Node.

Wenn du diesen Editor siehst, ist die Installation abgeschlossen.

## Update

1. Editor schließen.
2. `Plugins/MayDialogue/` durch die neue Version ersetzen (alten Ordner vorher sichern).
3. Projekt-Dateien neu generieren.
4. Rebuild.
5. Editor öffnen und Dialog-Assets im Content Browser prüfen. Bei Validator-Warnungen: [Migrations-Hinweise](../troubleshooting/common-issues.md) beachten.

## Deinstallation

1. Editor schließen.
2. `Plugins/MayDialogue/` löschen.
3. Projekt-Dateien neu generieren und rebuild.

{% hint style="danger" %}
**Achtung vor Orphan-Assets.** Dialog-Assets, die auf MayDialogue-Klassen verweisen, schlagen nach der Deinstallation fehl. Exportiere oder migriere sie, bevor du das Plugin entfernst.
{% endhint %}

## Nächster Schritt

[→ Quick Start](quick-start.md)
