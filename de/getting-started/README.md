---
description: In drei Schritten vom leeren Projekt zum spielbaren Dialog.
---

# Erste Schritte

MayDialogue ist ein Dialog-Plugin für Unreal Engine 5.7. Du bekommst einen visuellen Graph-Editor, eine fertige UI-Schicht, 3D-Audio mit prozeduralem Babel-Fallback, Kamerasteuerung, Typewriter-Effekte und GAS-Integration. Alles steckt in einem einzigen Plugin, ohne fremde Abhängigkeiten.

> 📸 **Bild-Platzhalter:** `getting-started-overview.png`: Screenshot des MayDialogue-Editors mit einem geöffneten Dialog-Asset.
> *Setup:* Asset `DA_Gate_Guardian` im Editor offen. Sichtbar: der Graph mit mehreren farbigen Nodes (Entry grün, SayLines dunkelrot/grau, PlayerChoice breit, Exit rot), das Speakers-Panel rechts mit zwei Einträgen, das Outline-Panel links mit Node-Liste. Bildunterschrift: "Der Graph ist das Dokument."

## Was du brauchst

* **Unreal Engine 5.7** (Binary- oder Source-Build)
* Ein bestehendes Projekt (Blueprint- oder C++-Projekt)
* Grundkenntnisse in Blueprint oder C++. Plugin-Interna musst du nicht kennen.

## Die drei Schritte

### 1: Installation

Plugin-Ordner ins Projekt kopieren, Projekt neu generieren, Editor öffnen. Dauert rund fünf Minuten.

[→ Installation](installation.md)

### 2: Quick Start

In fünf Minuten ein spielbarer NPC-Dialog im Level. Kein UMG-Setup, keine Audio-Konfiguration, kein Input-Handling.

[→ Quick Start](quick-start.md)

### 3: Walkthrough

Ein vollständiger Dialog mit Variablen, GAS-Attribut-Checks, Choice-Requirements und SideEffect-Aktionen.

[→ Walkthrough](first-dialogue.md)

## Empfohlene Reihenfolge

```
Installation → Quick Start → Walkthrough → Projekt-Einstellungen → Kern-Konzepte
```

Danach kannst du dich gezielt den Themen widmen, die dein Projekt braucht: [UI](../ui/README.md), [Audio](../audio/README.md), [GAS](../gas/README.md) oder [Rezepte](../recipes/README.md).

> 📸 **Bild-Platzhalter:** `getting-started-flow-diagram.png`: Übersichtsgrafik der empfohlenen Lernpfade.
> *Setup:* Einfache Grafik mit fünf Feldern von links nach rechts: "Installation" → "Quick Start" → "Walkthrough" → "Projekt-Einstellungen" → "Kern-Konzepte". Jedes Feld als beschriftete Box, Pfeile dazwischen.

{% hint style="warning" %}
**Nur Blueprint-Projekt?** Du kannst das Plugin als Binary-Distribution nutzen, ohne selbst zu kompilieren. Sobald du eigene Nodes, Requirements oder SideEffects in C++ schreibst, braucht dein Projekt ein C++-Modul und einen Rebuild.
{% endhint %}
