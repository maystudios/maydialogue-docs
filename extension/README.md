---
description: Was du erweitern kannst — Nodes, Requirements, SideEffects, Bridge.
---

# Erweiterung

MayDialogue ist darauf ausgelegt, dass du projekt-spezifische Logik direkt ins Plugin-Ökosystem einbringst. Alles, was du erweiterst, erscheint automatisch in den Editor-Pickern — kein Registrierungsschritt, kein Boilerplate.

## Was ist erweiterbar?

| Erweiterungspunkt | Wie | Wozu |
| --- | --- | --- |
| **Eigene Node-Typen** | Blueprint-Subklasse von `UMayDialogueNode_Base` | Projekt-spezifische Graph-Schritte (z.B. "Notify Quest System") |
| **Eigene Requirements** | Blueprint-Subklasse von `UMayDialogueRequirement` | Projekt-spezifische Bedingungen (z.B. "Quest abgeschlossen", "Item vorhanden") |
| **Eigene SideEffects** | Blueprint-Subklasse von `UMayDialogueSideEffect` | Projekt-spezifische Aktionen (z.B. "Achievement freischalten", "Quest-Progress +1") |
| **Bridge-Events binden** | Delegates des Subsystems abonnieren | Externes System auf Dialog-Events reagieren lassen |

## Wie Discovery funktioniert

UE's Reflection-System scannt alle `UClass`-Subklassen zur Laufzeit. Sobald eine Blueprint- oder C++-Klasse kompiliert ist, erscheint sie automatisch:

* In den **Requirement-Pickern** jeder Choice, jedes Branch, jeder SayLine.
* In den **SideEffect-Arrays** jedes Nodes.
* Im **Kontext-Menü** des Dialog-Graphen (für eigene Nodes).

Du registrierst nichts manuell.

> 📸 **Bild-Platzhalter:** `ext-overview-picker.png` — Dropdown-Menü "Add Requirement" mit eigener BP-Klasse in der Liste.
> *Setup:* MayDialogue-Editor offen, Choice-Node ausgewählt. Im Details-Panel: "Add Requirement"-Dropdown aufgeklappt. Liste zeigt Standard-Klassen (`Has Gameplay Tag`, `Check GAS Attribute`, …) und darunter die eigene Blueprint-Klasse `BP_Req_QuestCompleted` (unter Kategorie "Quest"). Letztere ist hervorgehoben.

## Blueprint-First-Prinzip

Alle Basisklassen sind `Blueprintable` und `EditInlineNew`. Das bedeutet:

* **Für 90% aller Erweiterungen reicht Blueprint.** Du klickst, nicht tippst.
* C++ ist sinnvoll, wenn du tiefe Integration in eigene C++-Systeme brauchst oder Performance-kritische Logik vorliegt.

Jede Seite dieses Abschnitts zeigt zuerst den Blueprint-Weg, dann eine C++-Variante. Die [C++-Grundlagen](cpp-fundamentals.md) (Build.cs-Setup, BlueprintNativeEvent-Pattern, Module-Reload) sind einmalig dokumentiert und für alle drei Erweiterungs-Typen gleich.

## Kapitel im Detail

* [Eigene Nodes](custom-nodes.md) — Neue Graph-Schritte mit eigenem Verhalten.
* [Eigene Requirements](custom-requirements.md) — Neue Bedingungen für Choices, Branches, SayLines.
* [Eigene SideEffects](custom-side-effects.md) — Neue Inline-Aktionen an beliebigen Nodes.
* [Bridge-Implementation](bridge-implementation.md) — Dialog-Events in externe Systeme leiten.
* [C++-Erweiterung — Grundlagen](cpp-fundamentals.md) — Build.cs, BlueprintNativeEvent, Module-Reload, Live-Coding-Caveats.

> 📸 **Bild-Platzhalter:** `ext-overview-architecture.png` — Diagramm: Basisklassen-Hierarchie mit Blueprint-Subklassen als Erweiterungspunkte.
> *Setup:* Einfaches UML-ähnliches Diagramm. Vier Basisklassen-Boxen: `UMayDialogueNode_Base`, `UMayDialogueRequirement`, `UMayDialogueSideEffect`, `IMayDialogueBridge`. Darunter jeweils Beispiel-Subklassen als gestrichelte Boxen: `BP_DN_NotifyQuest`, `BP_Req_QuestCompleted`, `BP_SE_QuestProgress`, `UMyQuestBridgeConsumer`. Pfeile von Subklassen zu Basisklassen.

## Best Practices — kurz

* **Eine Klasse, eine Aufgabe.** `BP_Req_QuestActive` prüft Quest-Status — und nichts anderes.
* **Display-Namen setzen.** In Class Settings einen verständlichen Namen eintragen — er erscheint in Pickern und Pill-Labels.
* **Null-Guards einbauen.** Prüfe, ob dein Subsystem vorhanden ist. Gib bei fehlendem Subsystem `Passed` zurück (Requirement) oder tue nichts (SideEffect) — nie crashen.
* **Fail-Modi respektieren.** `bHideOnFail` in Requirements sollte Designer-steuerbar bleiben.

> 📸 **Bild-Platzhalter:** `ext-overview-pill-labels.png` — Graph-Ansicht mit eigenen SideEffect-Pills an einer SayLine, gut lesbare Display-Namen.
> *Setup:* MayDialogue-Editor, SayLine-Node aufgeklappt. SideEffects-Array zeigt drei Pills: "Quest Progress +1 (KillDragon)", "Grant Achievement: DragonSlayer", "Add Tag: Story.DragonDefeated". Pill-Labels kommen aus `GetDisplayDescription`-Rückgabe der jeweiligen Klassen.
