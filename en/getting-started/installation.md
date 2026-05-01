---
description: Adding the plugin to your project — step by step.
---

# Installation

MayDialogue is a UE plugin with three modules. Installation runs in five steps and takes about five minutes.

## Prerequisites

| Component | Minimum version | Note |
| --- | --- | --- |
| Unreal Engine | 5.7 | Binary or source build |
| Visual Studio 2022 | 17.9+ | Only if your project has a C++ module |
| JetBrains Rider | 2024.3+ | Optional |

**Plugin dependencies** (enabled automatically, no manual activation needed):

* `GameplayAbilities`
* `GameplayTagsEditor`
* `StructUtils`
* `EnhancedInput`

## Step 1: Copy the plugin folder

1. Close the Unreal Editor if it is running.
2. Navigate to the `Plugins/` folder in your project directory. If it does not exist, create it.
3. Copy the entire `MayDialogue` folder (including `MayDialogue.uplugin`) to `Plugins/MayDialogue/`.

Your directory structure afterwards:

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

> 📸 **Image placeholder:** `installation-folder-structure.png` — Windows Explorer with the project's `Plugins/` folder open.
> *Setup:* Explorer window showing `Plugins/MayDialogue/` with subfolders `Source/`, `Content/`, `Resources/`, and the file `MayDialogue.uplugin`. Red arrow pointing at `MayDialogue.uplugin`.

## Step 2: Regenerate project files

Only required if your project has a C++ module (`.uproject` contains `"Modules"` with at least one entry).

{% tabs %}
{% tab title="Windows" %}
Right-click on `.uproject` → **Generate Visual Studio project files**, or in a shell:

```bash
"C:/Program Files/Epic Games/UE_5.7/Engine/Build/BatchFiles/GenerateProjectFiles.bat" \
  -project="C:/Path/To/MyProject.uproject" -game -rocket
```
{% endtab %}

{% tab title="macOS / Linux" %}
```bash
./UnrealEngine/GenerateProjectFiles.sh -project=/path/to/MyProject.uproject -game
```
{% endtab %}
{% endtabs %}

## Step 3: Build the project

Open the `.sln` in Rider or Visual Studio and build the **Editor target** (`MyProjectEditor`, Development, Win64).

Alternatively, via batch file:

```bash
"C:/Program Files/Epic Games/UE_5.7/Engine/Build/BatchFiles/Build.bat" \
  MyProjectEditor Win64 Development "-project=C:/Path/To/MyProject.uproject"
```

{% hint style="info" %}
**Blueprint-only projects** can skip this step. UE will automatically compile the necessary binaries on first editor launch.
{% endhint %}

## Step 4: Open the editor and verify the plugin

1. Open the project.
2. Go to **Edit → Plugins**.
3. Select the **Gameplay** category. The **MayDialogue** entry must be visible and enabled (checkbox checked).

> 📸 **Image placeholder:** `installation-plugin-dialog.png` — The Plugins dialog with the MayDialogue entry.
> *Setup:* Plugins window open, "Gameplay" category selected. MayDialogue entry visible with an enabled checkbox, the beta banner below it, and the plugin description text. Red arrow pointing at the enabled checkbox.

{% hint style="info" %}
**Beta flag.** MayDialogue is marked as `IsBetaVersion: true`. UE shows a warning banner in the Plugins dialog. You can safely ignore it.
{% endhint %}

## Step 5: Enable plugin content

MayDialogue ships a content directory with default assets. To make the Content Browser show it:

1. Open the **Content Browser**.
2. Click the small **Settings icon** in the top right.
3. Enable **Show Plugin Content**.

## Verify the installation

Create a folder `Content/Dialogues/` and do the following:

1. Right-click in the Content Browser → **May Dialogue → Dialogue Asset**.
2. Name the asset: `DA_Test_Install`.
3. Double-click it. The asset editor opens with an empty graph and an **Entry** node.

If you see this editor, the installation is complete.

## Updating

1. Close the editor.
2. Replace `Plugins/MayDialogue/` with the new version (back up the old folder first).
3. Regenerate project files.
4. Rebuild.
5. Open the editor and check dialogue assets in the Content Browser. If you see validator warnings, consult the [migration notes](../troubleshooting/common-issues.md).

## Uninstalling

1. Close the editor.
2. Delete `Plugins/MayDialogue/`.
3. Regenerate project files and rebuild.

{% hint style="danger" %}
**Watch out for orphaned assets.** Dialogue assets that reference MayDialogue classes will fail after uninstallation. Export or migrate them before removing the plugin.
{% endhint %}

## Next step

[→ Quick Start](quick-start.md)
