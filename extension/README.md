# Erweiterung

MayDialogue ist als **Baustein** gedacht, auf dem andere Systeme aufsetzen. Dieser Abschnitt zeigt, wie du das Plugin projekt-spezifisch erweiterst.

## Was kann man erweitern?

| Erweiterungspunkt | Wie |
| --- | --- |
| Eigene Node-Typen | Blueprint-Subklasse von `UMayDialogueNode_Base`. |
| Eigene Requirements | Blueprint-Subklasse von `UMayDialogueRequirement`. |
| Eigene SideEffects | Blueprint-Subklasse von `UMayDialogueSideEffect`. |
| Eigene Choices | Blueprint-Subklasse von `UMayDialogueChoice`. |
| Eigene Bridge-Implementationen | C++-Klasse implementiert `IMayDialogueBridge`. |
| Eigene UI-Widget-Skins | Blueprint-Subklassen der Widget-Klassen. |
| Eigene Babel-Profile | DataAsset von `UMayDialogueBabelProfile`. |

## Kapitel-Überblick

* [Eigene Nodes per Blueprint](custom-nodes.md)
* [Eigene Requirements](custom-requirements.md)
* [Eigene SideEffects](custom-side-effects.md)
* [Bridge implementieren](bridge-implementation.md)

## Philosophie: Blueprint-First für Designer

Alle Basis-Klassen sind `Blueprintable` + `EditInlineNew`. Designer müssen für 90% aller Erweiterungen **kein C++ schreiben**.

C++ ist nur sinnvoll, wenn:

* Die Logik komplex ist und Performance-kritisch.
* Die Klasse tief in andere C++-Systeme (Quest, Inventory) eingreift.
* Du native Tags oder Interfaces zur Compile-Time-Typ-Sicherheit haben willst.

## Reflection-basierte Discovery

UE's Reflection-System findet **alle Subklassen** der Basis-Klassen automatisch. Sobald eine Blueprint-Klasse kompiliert ist, erscheint sie in den Node-/Sub-Node-Pickern. Kein Registrierungs-Schritt nötig.

## Best Practices

* **Kleine, fokussierte Klassen**. Ein Requirement pro Regel, ein SideEffect pro Aktion. Lieber mehr Klassen, jede klar benannt.
* **Display-Namen setzen**. `UCLASS(meta = (DisplayName = "..."))` oder Blueprint-Node-Title-Override.
* **Description-Felder nutzen**. `Description` ist die Pill-Label-Quelle im Graph.
* **Fail-Modi respektieren**. `bHideOnFail` und `FailureResult` sollten Designer-steuerbar bleiben.
