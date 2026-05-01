---
description: Großen Dialog mit SubGraph-Nodes in überschaubare Abschnitte aufteilen – ohne Assets zu splitten.
---

# SubGraph-Organisation

## Szenario

Dein Hauptquest-NPC hat einen langen Dialog mit Intro, zwei Gesprächs-Themen und einem Abschluss-Block. Der Graph wächst auf 30+ Nodes und wird unübersichtlich. Mit SubGraphs kollabierst du jeden Block in eine einzelne Box – der Hauptgraph bleibt lesbar wie ein Inhaltsverzeichnis.

## Was du lernst

- SubGraph-Node anlegen und befüllen.
- Mit Breadcrumb-Navigation zwischen Haupt- und Sub-Graph wechseln.
- SubGraph-Scope: Variablen und Speaker erben vom Parent.
- Den Unterschied zwischen SubGraph (intern) und Link (extern).

## Voraussetzungen

- [Einfaches NPC-Gespräch](simple-npc-talk.md) abgeschlossen.
- Verständnis des [Link-Nodes](linking-dialogues.md).

## Mini-Graph

```text
[Entry]
   │
   ▼
[SubGraph: "Intro"]
   │  (intern: SayLine × 2 → Exit)
   ▼
[SubGraph: "Thema A: Das Artefakt"]
   │  (intern: SayLine → PlayerChoice → Branch → ...)
   ▼
[SubGraph: "Thema B: Die Bedrohung"]
   │  (intern: Branch → SayLine × 3 → ...)
   ▼
[SubGraph: "Abschluss"]
   │  (intern: SayLine → AddTag → Exit)
   ▼
[Exit: Completed]
```

> 📸 **Bild-Platzhalter:** `subgraph-organization-main-graph.png` — Hauptgraph mit vier SubGraph-Boxes.
> *Setup:* Asset `DA_QuestNPC_Main` geöffnet. Sichtbar: Entry → vier SubGraph-Nodes untereinander (Ordner-Icon in Title-Bar, Namen lesbar) → Exit. Jede Box kompakt, keine internen Nodes sichtbar. Der Graph passt komplett in den Viewport.

## Schritt-für-Schritt

### 1. Asset mit langen Nodes anlegen

Asset: `DA_QuestNPC_Main`. Alle 30 Nodes zunächst flach im Graph anlegen und testen.

### 2. Ersten SubGraph anlegen

Rechtsklick im Graph → **Create Node → SubGraph**. Name im Title: *„Intro"*. Der SubGraph-Node erscheint als einzelne Box mit Ordner-Icon.

### 3. Nodes in den SubGraph verschieben

Doppelklick auf den SubGraph-Node → neuer Tab öffnet sich mit eigenem Entry + Exit. Kopiere die Intro-Nodes aus dem Hauptgraph via **Ctrl+C / Ctrl+V** hinein. Verbinde sie mit dem SubGraph-internen Entry und Exit. Lösche die Original-Nodes im Hauptgraph.

> 📸 **Bild-Platzhalter:** `subgraph-organization-subgraph-inside.png` — Geöffneter SubGraph "Intro" mit internen Nodes.
> *Setup:* Tab "Intro" oben mit Breadcrumb `DA_QuestNPC_Main > Intro`. Im Graph: SubGraph-Entry (kleiner grüner Node) → zwei SayLines → SubGraph-Exit. Breadcrumb-Bar am oberen Rand des Graph-Panels sichtbar.

### 4. Breadcrumb-Navigation

Oben im Graph-Panel erscheint die Breadcrumb-Leiste: `DA_QuestNPC_Main > Intro`. Klick auf `DA_QuestNPC_Main` kehrt zum Hauptgraph zurück.

### 5. Restliche Blocks auslagern

Wiederhole Schritt 2–3 für *Thema A*, *Thema B* und *Abschluss*. Verbinde die SubGraph-Nodes im Hauptgraph in der richtigen Reihenfolge.

### 6. SubGraph-Namen vergeben

Jeder SubGraph hat ein `Description`-Feld im Details-Panel. Nutze präzise Namen: *„Intro – Begrüßung"*, *„Thema A – Artefakt-Frage"*. Diese Namen erscheinen auch im Outline-Panel.

> 📸 **Bild-Platzhalter:** `subgraph-organization-outline-panel.png` — Outline-Panel mit aufgeklappten SubGraph-Einträgen.
> *Setup:* Outline-Panel links geöffnet. Einträge: SubGraph "Intro" mit Child-Nodes darunter (SayLine × 2), SubGraph "Thema A" mit Kindern usw. Farb-Chips der Speaker sichtbar.

## SubGraph vs. Link – Kurzvergleich

| | SubGraph | Link |
|--|----------|------|
| Lebt in | Demselben Asset | Eigenem Asset |
| Wiederverwendbar | Nein | Ja |
| Speaker-Scope | Erbt vom Parent | Eigener Scope |
| Für | Interne Struktur | Geteilte Fragmente |

## Blueprint-Triggering

Kein besonderer Code. SubGraphs sind intern – nach außen startet der Dialog ganz normal:

```text
[Event OnInteract]
   │
   ▼
[MayDialogueLibrary → Start Dialogue]
   ├─ Asset: DA_QuestNPC_Main
   └─ ...
```

## Variation / Weiter gehen

- SubGraph **bedingt überspringen**: Statt direkter Verbindung einen Branch davor – je nach Quest-Status einen SubGraph überspringen.
- Für echte Wiederverwendung SubGraph zu einem eigenen Asset auslagern und per [Link](linking-dialogues.md) referenzieren.
- Im Outline-Panel mit **Filter by SubGraph** schnell zwischen Abschnitten navigieren.

## Troubleshooting

**SubGraph-Node hat keine Output-Verbindung.**
Der interne SubGraph-Exit muss mit dem SubGraph-internen Exit-Node verbunden sein – nicht mit dem Haupt-Exit-Node. Doppelklick → internen Exit prüfen.

**Variablen im SubGraph sichtbar, aber Werte falsch.**
SubGraph erbt den Scope des Parent-Assets. Wenn du eine Variable im SubGraph setzt und sie danach im Hauptgraph liest, ist das derselbe Scope – funktioniert korrekt. Prüfe Tippfehler im Variablen-Namen.

**Breadcrumb zeigt falsche Tiefe.**
Mehrfach verschachtelte SubGraphs sind möglich – der Breadcrumb zeigt die gesamte Tiefe. Navigiere mit Klick auf beliebigen Breadcrumb-Eintrag nach oben.
