---
description: >-
  Ein Dialog-Plugin für Unreal Engine 5.7 mit visuellem Graph-Editor,
  3D-Audio, Typewriter, Kamera-Steuerung, GAS-Integration und einem fertigen
  UI-Set – sofort einsetzbar, ohne Boilerplate.
---

# Willkommen bei MayDialogue

**MayDialogue** ist ein vollständiges Dialogsystem für Unreal Engine 5.7. Du musst kein Audio-Routing aufsetzen, keine UMG-Hierarchie bauen, keine Input-Mode-Logik schreiben. Plugin aktivieren, NPC mit einer Komponente versehen, Dialog-Asset zuweisen, Start-Funktion aufrufen — Audio läuft in 3D, das Widget erscheint, Text läuft per Typewriter, Choices sind klickbar, der Dialog endet sauber.

{% hint style="success" %}
**Fünf Minuten von Null bis spielbar.** Kein Blueprint-Boilerplate, kein UI-Setup. Schau dir den [Quick Start](getting-started/quick-start.md) an.
{% endhint %}

> 📸 **Bild-Platzhalter:** `hero-running-dialogue.png` — Screenshot eines laufenden Dialogs im Spiel.
> *Setup:* PIE-Session in einem Test-Level. Sichtbar: ein NPC im Vordergrund, Dialog-Widget am unteren Bildschirmrand mit Sprecher-Name "Wächter", Portrait links, Text "Halt! Wer bist du?" mit aktivem Typewriter (mittendrin), zwei Choice-Buttons "Ein Freund des Königs." / "Das geht dich nichts an." Skip-Hinweis "Space drücken" am rechten Rand.

## Was du bekommst

<table data-view="cards">
  <thead>
    <tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th></tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Visueller Graph-Editor</strong></td>
      <td>Sprecher-Farben, Inline-Text, Sub-Nodes als Pills im Body — der Graph liest sich wie ein Drehbuch.</td>
      <td><a href="concepts/graph-visual-language.md">graph-visual-language.md</a></td>
    </tr>
    <tr>
      <td><strong>Sofort spielbares UI</strong></td>
      <td>Slate-Debug-Widget out-of-the-box; austauschbares UMG-Komponenten-Set für Production.</td>
      <td><a href="ui/README.md">UI-System</a></td>
    </tr>
    <tr>
      <td><strong>3D-Audio mit Fallback</strong></td>
      <td>Voice spielt automatisch am Sprecher-Actor. Drei Override-Ebenen (Plugin / Sprecher / Node).</td>
      <td><a href="audio/README.md">Audio-System</a></td>
    </tr>
    <tr>
      <td><strong>GAS-Integration</strong></td>
      <td>Tag-, Attribut- und Ability-Requirements; ApplyEffect, AddTag, RemoveTag, TriggerCue als Nodes.</td>
      <td><a href="gas/README.md">GAS-Integration</a></td>
    </tr>
    <tr>
      <td><strong>Preview ohne PIE</strong></td>
      <td>Dialog direkt aus dem Asset-Editor abspielen, mit simuliertem GAS-State und Culture-Switch.</td>
      <td><a href="editor/preview-runner.md">Preview-Runner</a></td>
    </tr>
    <tr>
      <td><strong>Robust und erweiterbar</strong></td>
      <td>Production-Architektur nach Epic Best-Practices. Eigene Nodes, Requirements und SideEffects per Blueprint.</td>
      <td><a href="extension/README.md">Erweitern</a></td>
    </tr>
  </tbody>
</table>

## Für wen ist das?

MayDialogue passt zu:

* **Singleplayer-Story-Spiele** — Horror, Visual Novel, RPG, Walking Simulator, narratives Adventure.
* **Indie-Teams und Game Jams** — in Minuten produktiv, kein Wochen-Setup.
* **Projekte, die GAS nutzen** — Tags, Attribute und Effects sprechen die gleiche Sprache wie deine Charaktere.
* **Beginner ohne C++-Erfahrung** — alle wichtigen Workflows funktionieren in Blueprint.
* **Erfahrene C++-Entwickler** — Subsystem-API, Delegates und C++-Subklassen für eigene Logik.

MayDialogue ist **kein**:

* Quest- oder Mission-System (lese Quest-Status über Tags / Variablen, der Quest-Tracker selbst lebt anderswo).
* Voice-Recording- oder TTS-Tool (das Plugin spielt fertige `USoundBase`-Assets ab).
* Sequencer-Ersatz (für längere Cutscenes startest du Level Sequences, die wiederum MayDialogue-Dialoge triggern können).

## Wo soll ich anfangen?

{% content-ref url="getting-started/quick-start.md" %}
[quick-start.md](getting-started/quick-start.md)
{% endcontent-ref %}

{% content-ref url="getting-started/first-dialogue.md" %}
[first-dialogue.md](getting-started/first-dialogue.md)
{% endcontent-ref %}

{% content-ref url="recipes/README.md" %}
[recipes/](recipes/README.md)
{% endcontent-ref %}

## Lese-Pfade

Such dir den Pfad, der zu dir passt:

| Du bist… | Empfohlene Reihenfolge |
| --- | --- |
| **Beginner ohne C++** | [Installation](getting-started/installation.md) → [Quick Start](getting-started/quick-start.md) → [Walkthrough](getting-started/first-dialogue.md) → [Rezepte](recipes/README.md) |
| **Designer** | [Quick Start](getting-started/quick-start.md) → [Graph & visuelle Sprache](concepts/graph-visual-language.md) → [Editor-Tour](editor/README.md) → [Rezepte](recipes/README.md) |
| **Blueprint-Entwickler** | [Walkthrough](getting-started/first-dialogue.md) → [Runtime-Integration](runtime/README.md) → [Bridge & Events](runtime/bridge-events.md) → [Eigene Nodes](extension/custom-nodes.md) |
| **C++-Entwickler** | [Architektur](concepts/architecture.md) → [Subsystem-API](runtime/subsystem-api.md) → [Reference](reference/README.md) → [Erweiterung](extension/README.md) |
| **UI-Designer** | [UI-Übersicht](ui/README.md) → [UMG-Architektur](ui/umg-architecture.md) → [Themes](ui/themes.md) → [Rich-Text-Tags](ui/rich-text-tags.md) |

## Status

| Information | Wert |
| --- | --- |
| Engine | Unreal Engine 5.7 |
| Plugin-Version | 0.1.0 (Beta) |
| Letzte Aktualisierung | 2026-04-27 |
| Module | `MayDialogue` (Runtime) · `MayDialogueEditor` (UncookedOnly) · `MayDialogueGAS` (Runtime) |
| Dependencies | `GameplayAbilities`, `GameplayTags`, `StructUtils`, `EnhancedInput` |

Aktuelle Limitierungen und geplante Features findest du unter [Bekannte Issues](troubleshooting/known-issues.md) und [Roadmap](appendix/roadmap.md).

---

> 📸 **Bild-Platzhalter:** `hero-graph-overview.png` — Übersichtsfoto eines kompletten Dialog-Graphen.
> *Setup:* Beispiel-Asset `DA_Demo_VillagerEncounter` im Asset-Editor offen. Auto-Layout angewandt. Sichtbar: Entry-Node grün links, drei Sprecher-gefärbte SayLines (zwei rote Wächter-Zeilen, eine graue Spieler-Zeile), ein PlayerChoice-Node mit drei Choice-Sub-Nodes, ein Branch-Node mit Requirement-Sub-Node "HasTag(Story.Found.Codex)", zwei Reaktions-SayLines, ein Exit-Node rot rechts. Comment-Box "Begrüßung" um die ersten drei Nodes. Outline-Tab links sichtbar mit der Node-Liste.
