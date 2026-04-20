---
description: >-
  Ein designer-zentriertes Dialog-Plugin für Unreal Engine 5.7 mit visuellem
  Graph-Editor, integrierter UI, 3D-Audio, Typewriter-Effekten, Kamera-Steuerung
  und voller GAS-Integration.
---

# Willkommen bei MayDialogue

**MayDialogue** ist ein vollständiges, sofort einsetzbares Dialogsystem für Unreal Engine 5.7. Statt dich wochenlang mit Text-Rendering, Audio-Pipelines und Branching-Logik herumzuschlagen, lieferst du deine Dialog-Erfahrung wörtlich in wenigen Minuten: NPC-Actor ins Level setzen, Dialog-Asset zuweisen, fertig.

{% hint style="success" %}
**Fünf Minuten, kein Blueprint-Boilerplate.** Plugin aktivieren, `MayDialogueParticipant` an einen NPC hängen, Dialog-Asset zuweisen, `StartDefaultDialogue` aufrufen — Audio spielt in 3D, Widget taucht auf, Text läuft per Typewriter, Choices sind klickbar, Dialog endet sauber, Kamera kehrt zurück.
{% endhint %}

## Was MayDialogue auszeichnet

<table data-view="cards">
  <thead>
    <tr><th></th><th></th><th data-hidden data-card-target data-type="content-ref"></th></tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Designer-First-Graph</strong></td>
      <td>Jeder Node zeigt Sprecher, Text und Sub-Aktionen inline. Kein Panel-Hopping, kein Property-Jagen.</td>
      <td><a href="concepts/graph-visual-language.md">graph-visual-language.md</a></td>
    </tr>
    <tr>
      <td><strong>Batteries-Included-UI</strong></td>
      <td>Slate-Debug-Widget für Prototyping, komponenten-basiertes UMG für Production. Beides out-of-the-box.</td>
      <td><a href="ui/README.md">UI-System</a></td>
    </tr>
    <tr>
      <td><strong>3D-First-Audio</strong></td>
      <td>Voice kommt automatisch aus dem Sprecher-Actor. 3-Level-Fallback (Plugin → Sprecher → Node) für volle Kontrolle.</td>
      <td><a href="audio/README.md">Audio-System</a></td>
    </tr>
    <tr>
      <td><strong>Volle GAS-Integration</strong></td>
      <td>Requirements und Action-Nodes für Tags, Attribute, Effects und Cues – erste-Klasse-Bürger, kein Add-on.</td>
      <td><a href="gas/README.md">GAS-Integration</a></td>
    </tr>
    <tr>
      <td><strong>In-Editor-Preview</strong></td>
      <td>Dialog direkt aus dem Asset-Editor abspielen – ohne PIE, mit simuliertem GAS-State und Culture-Switch.</td>
      <td><a href="editor/preview-runner.md">preview-runner.md</a></td>
    </tr>
    <tr>
      <td><strong>CommonConversation-Core</strong></td>
      <td>Robuste Laufzeit-Architektur aus Epics CC-Plugin (Lyra) – ohne externe Plugin-Dependency.</td>
      <td><a href="concepts/common-conversation.md">common-conversation.md</a></td>
    </tr>
  </tbody>
</table>

## Ist MayDialogue das Richtige für dich?

MayDialogue ist optimiert für:

* **Singleplayer-Story-Spiele** – Horror, Visual Novel, RPG, Walking Simulator.
* **Multiplayer-fähigen Core** – Server/Client-RPC-Pfade folgen dem CC-Pattern; nachrüstbar.
* **Game-Jam- und Indie-Teams** – in Minuten produktiv, kein Wochen-Setup.
* **Projekte, die GAS nutzen** – Dialoge lesen und schreiben Tags, Attribute, Effects nativ.

MayDialogue ist **kein**:

* Quest- oder Mission-System (siehe [Non-Goals](appendix/common-conversation-comparison.md#kein-quest-system)).
* Voice-Recording- oder TTS-Tool (das Plugin spielt fertige `USoundBase`-Assets ab).
* Sequencer-Ersatz (für komplexe Cutscenes verwendest du Level Sequences, die MayDialogue-Dialoge starten können).

## Schnell einsteigen

{% content-ref url="getting-started/quick-start.md" %}
[quick-start.md](getting-started/quick-start.md)
{% endcontent-ref %}

{% content-ref url="getting-started/first-dialogue.md" %}
[first-dialogue.md](getting-started/first-dialogue.md)
{% endcontent-ref %}

{% content-ref url="recipes/README.md" %}
[README.md](recipes/README.md)
{% endcontent-ref %}

## Die vier Design-Prinzipien

Jede Entscheidung in MayDialogue folgt vier Leitsätzen. Wer das Plugin verstehen will, sollte sie kennen.

1. **Designer-First.** Der Graph ist das Dokument. Wer spricht, was sagt er, welche Wahl hat der Spieler – alles ohne Panel-Wechsel lesbar.
2. **Batteries-Included.** UI, Audio, Kamera, Typewriter – alles mitgeliefert. Dein Projekt hat am Tag 1 ein spielbares Dialogsystem, nicht nur ein Framework.
3. **CommonConversation-Inspiriert.** Die Laufzeit-Architektur spiegelt Epics interne Lösung (Instance-Lifecycle, Participant-Component, Scope-Stack) – ohne CC als Dependency einzuziehen.
4. **Singleplayer-First, Multiplayer-Ready.** Heute läuft dein Horror-Solo-Game, morgen bleibt die Tür für Multiplayer offen.

Mehr Details: [Kern-Konzepte → Architektur](concepts/architecture.md).

## Status & Versionen

| Information | Wert |
| --- | --- |
| Engine | Unreal Engine 5.7 |
| Plugin-Version | 0.1.0 (Beta) |
| Letzte Aktualisierung | 2026-04-20 |
| Module | `MayDialogue` (Runtime) · `MayDialogueEditor` (UncookedOnly) · `MayDialogueGAS` (Runtime) |
| Dependencies | `GameplayAbilities`, `GameplayTags`, `StructUtils`, `EnhancedInput` |

Du findest die aktuelle Roadmap und bekannte Issues unter [Troubleshooting → Bekannte Issues](troubleshooting/known-issues.md).

---

{% hint style="info" %}
**Diese Dokumentation ist die Benutzer-Anleitung.** Die interne Design-Spezifikation des Plugins lebt in `Plugins/MayDialogue/README.md` und richtet sich an Plugin-Entwickler. Wenn du *das Plugin benutzt*, ist diese GitBook-Doku dein Ort.
{% endhint %}
