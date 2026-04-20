# Installation

MayDialogue ist ein gewöhnliches UE-Plugin mit drei Modulen und vier Dependency-Plugins. Die Installation dauert rund fünf Minuten.

## Voraussetzungen

| Komponente | Mindestversion | Hinweis |
| --- | --- | --- |
| Unreal Engine | 5.7 | Binary oder Source |
| Visual Studio 2022 | 17.9+ | Nur wenn dein Projekt C++ enthält |
| JetBrains Rider | 2024.3+ | Optional – das Projekt benutzt Rider |
| Perforce / Git | – | Für Versionierung empfohlen |

**Plugin-Dependencies** (werden automatisch mitaktiviert):

* `GameplayAbilities`
* `GameplayTagsEditor`
* `StructUtils`
* `EnhancedInput`

## Schritt 1: Plugin in das Projekt kopieren

1. Schließe den Unreal-Editor, falls er läuft.
2. Navigiere in deinem Projekt-Ordner zu `Plugins/`. Falls der Ordner nicht existiert, lege ihn an.
3. Kopiere den gesamten `MayDialogue`-Ordner (inklusive `MayDialogue.uplugin`) nach `Plugins/MayDialogue/`.

Dein Projekt sollte jetzt so aussehen:

```
MyProject/
├── MyProject.uproject
├── Source/
├── Content/
└── Plugins/
    └── MayDialogue/
        ├── MayDialogue.uplugin
        ├── Source/
        ├── Content/
        └── Resources/
```

## Schritt 2: Projekt-Dateien neu generieren

Nur wenn dein Projekt einen C++-Code-Modul hat:

{% tabs %}
{% tab title="Windows" %}
Rechtsklick auf `.uproject` → **Generate Visual Studio project files**, oder in einer Bash-Shell:

```bash
"C:/Program Files/Epic Games/UE_5.7/Engine/Build/BatchFiles/GenerateProjectFiles.bat" \
  -project="C:/Pfad/Zu/MyProject.uproject" -game -rocket
```
{% endtab %}

{% tab title="macOS / Linux" %}
```bash
./UnrealEngine/GenerateProjectFiles.sh -project=/pfad/zu/MyProject.uproject -game
```
{% endtab %}
{% endtabs %}

## Schritt 3: Projekt bauen

Öffne die `.sln` in Rider oder Visual Studio und baue das **Editor-Target** (z.B. `MyProjectEditor`, Development, Win64).

Alternativ per Batch:

```bash
"C:/Program Files/Epic Games/UE_5.7/Engine/Build/BatchFiles/Build.bat" \
  MyProjectEditor Win64 Development "-project=C:/Pfad/Zu/MyProject.uproject"
```

## Schritt 4: Editor öffnen und Plugin prüfen

Öffne das Projekt. Beim ersten Start kompiliert UE Shader und das MayDialogue-Module. Anschließend:

1. **Edit → Plugins** öffnen.
2. Kategorie **Gameplay** wählen.
3. Der Eintrag **MayDialogue** muss sichtbar und **aktiviert** sein.

{% hint style="info" %}
**Beta-Flag.** MayDialogue ist als `IsBetaVersion: true` markiert. UE zeigt einen Warn-Banner im Plugin-Dialog; du kannst ihn ignorieren.
{% endhint %}

## Schritt 5: Content-Pfad freigeben

MayDialogue liefert ein Content-Verzeichnis mit (`Plugins/MayDialogue/Content/`). Damit die Default-Blip-Sounds und Beispiel-Assets gefunden werden, muss der Content-Browser das Plugin anzeigen:

1. **Content Browser** öffnen.
2. Kleines **Settings-Icon** rechts oben.
3. **Show Plugin Content** aktivieren.
4. **Show Engine Content** aktivieren (nur wenn du interne UE-Widgets inspizieren willst).

## Schritt 6: Installation prüfen

Erstelle einen neuen Asset-Ordner (`Content/Dialogues/`) und klicke:

1. Rechtsklick im Content Browser.
2. **Miscellaneous → MayDialogue Asset**.

Wenn der Eintrag existiert, ist die Installation erfolgreich. Benenne das Asset `DA_Test_Install`, doppelklick öffnet den Asset-Editor. Du siehst einen leeren Graph mit einem **Entry**-Node.

## Update

Beim Aktualisieren auf eine neue Version:

1. Editor schließen.
2. `Plugins/MayDialogue/` durch die neue Version ersetzen (alten Ordner sichern).
3. Projekt-Dateien neu generieren.
4. Rebuild.
5. Editor öffnen, die Asset-Dialoge im Content Browser prüfen – wenn Validator-Warnungen auftauchen, beachte das [Migrations-Kapitel](../troubleshooting/common-issues.md).

## Deinstallation

1. Editor schließen.
2. `Plugins/MayDialogue/` löschen.
3. Projekt-Dateien neu generieren, rebuild.

{% hint style="danger" %}
**Achtung vor Orphan-Assets.** Dialog-Assets, die auf Klassen aus MayDialogue verweisen, schlagen nach der Deinstallation fehl. Exportiere sie oder migriere sie zuerst, bevor du das Plugin entfernst.
{% endhint %}

## Nächster Schritt

Installation fertig? Direkt weiter mit dem [Quick Start](quick-start.md).
