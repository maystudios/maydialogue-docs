---
description: Schnelles Nachschlagen — Properties, Signaturen, Enums, Structs.
---

# Referenz

Der Referenz-Bereich ist ein Nachschlagewerk. Keine Erklärungen, keine Geschichten — nur die nackten Fakten: Property-Tabellen, Funktions-Signaturen, Enum-Werte.

## Was du hier findest

| Seite | Inhalt | Wann aufschlagen |
|---|---|---|
| [Projekt-Einstellungen](project-settings.md) | Alle Felder von `UMayDialogueSettings` mit Typ, Default und Bedeutung. | Setting-Name vergessen oder Default-Wert unklar. |
| [Editor-Einstellungen](editor-settings.md) | Alle Felder von `UMayDialogueEditorSettings` — Node-Farben, Debug-Highlights. | Node-Farben anpassen, Team-Defaults festlegen. |
| [API: Library](api-library.md) | Alle Methoden der `UMayDialogueLibrary` als Tabelle mit Parametern und Rückgabe. | Blueprint-Schnellstart, Signatur nachschlagen. |
| [API: Subsystem](api-subsystem.md) | Public-Methoden und Delegates des `UMayDialogueSubsystem`. | C++-Integration, Event-Binding, Lifecycle-Steuerung. |
| [API: Delegates](api-delegates.md) | Alle Delegate-Typen mit Signatur und Feuer-Zeitpunkt. | Game-System an Dialog-Phasen andocken. |
| [Typen & Enums](types.md) | Alle Enums und wichtigen Structs — Werte und Ein-Satz-Erklärung. | Auto-Complete-Rückschläge klären, Struct-Felder nachschlagen. |

## Was nicht hier steht

- **Konzepte & Architektur** → [Kern-Konzepte](../concepts/README.md)
- **Node-Dokumentation** → [Node-Referenz](../nodes/README.md)
- **GAS-spezifische Requirements/Actions** → [GAS-Integration](../gas/README.md)
- **UI-Widget-APIs** → [UI-System](../ui/README.md)

## Wie liest man die Property-Tabellen?

Überall dasselbe Schema:

| Property | Typ | Default | Bedeutung |
|---|---|---|---|
| `PropertyName` | `TTyp` | `Wert` | Ein Satz, was das Setting steuert. |

Typ-Konventionen:

- `TSoftObjectPtr<T>` / `TSoftClassPtr<T>` — Lazy-Referenz, wird erst beim ersten Dialog-Start geladen.
- `FGameplayTag` / `FGameplayTagContainer` — Tags aus dem UE-GameplayTags-System.
- Kein Default angegeben — zero-initialized bzw. `nullptr`.

## Welche Klasse ist wofür zuständig?

```text
UMayDialogueLibrary          (Blueprint-Helfer, stateless)
    │ delegiert an
UMayDialogueSubsystem        (Orchestrator, eine Instanz pro Welt)
    │ erzeugt und verwaltet
UMayDialogueInstance         (laufendes Gespräch, hat alle Delegates)
    │ liest aus
UMayDialogueAsset            (der Dialog-Graph, unveränderliche Blaupause)

UMayDialogueParticipant      (Actor-Komponente, bindet Actor an Instance)
    │ sitzt auf
AActor (Spieler, NPC, ...)

IMayDialogueBridge           (Interface, implementiert vom Subsystem)
```

> 📸 **Bild-Platzhalter:** `reference-class-overview.png` — Klassendiagramm als Annotationsdiagramm.
> *Setup:* Kein Editor nötig — handgezeichnetes oder Whiteboard-Foto des Diagramms oben. Klassen-Boxen beschriftet mit Klassenname + Kurzbeschreibung. Pfeile zeigen "delegiert an", "erzeugt", "sitzt auf", "implementiert". Farben: Library=Blau, Subsystem=Orange, Instance=Grün, Asset=Grau, Participant=Lila, Bridge=Gestrichelter Rahmen.
