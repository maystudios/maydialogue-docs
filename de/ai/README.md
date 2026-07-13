---
description: "MayDialogues Editor-MCP-Tools und ausgelieferte Dokumentation mit einem externen KI-Client verwenden."
---

# KI-Clients und MCP

MayDialogue liefert für Unreal Engine 5.8 eine ausschließlich im Editor verfügbare Model-Context-Protocol-Integration (MCP) aus. Ein MCP-fähiger Client kann damit Dialog-Assets, Babel-Profile, Lokalisierungsdaten und das versionierte MayDialogue-Dokumentationspaket über typisierte Tools untersuchen und bearbeiten.

{% hint style="info" %}
**MayDialogue enthält, trainiert und betreibt kein KI-Modell.** Der externe Client stellt sein eigenes Modell bereit und kombiniert es mit der Dokumentation und den Tools des laufenden Unreal Editors.
{% endhint %}

Die MCP-Integration ist bewusst Editor-only. Sie wird in Shipping-Builds weder geladen noch paketiert und fügt dem Spiel keine KI-Runtime-Abhängigkeit hinzu.

## Was die Dokumentations-Tools bereitstellen

Das ausgelieferte Paket wird aus denselben zweisprachigen Seiten erzeugt, die dieses GitBook speisen. Jedes Tool-Ergebnis nennt Plugin-Version, Dokumentations-Commit, Bundle-Content-Hash, Sprache und Quellseite beziehungsweise -abschnitt.

Ein MCP-Client kann:

* die englische oder deutsche Dokumentation durchsuchen;
* eine vollständige Seite oder einen benannten Abschnitt lesen;
* den kanonischen Fünf-Minuten-Quick-Start abrufen;
* eine stabile `MDV.*`-Validator-Diagnose erklären;
* aktuelle Einrichtungsanweisungen für Codex, Claude Code oder OpenCode abrufen.

Die Tools liefern echte ausgelieferte Dokumentation, keine getrennte Sammlung fest eingebauter Marketingantworten. Eine deterministische Drift-Prüfung schlägt in der Entwicklung fehl, sobald das generierte Paket nicht mehr zu den kanonischen Docs passt.

## Sicherheitsmodell

Dokumentations-Tools sind schreibgeschützt. Authoring-Tools verwenden strukturierte Requests, validieren Eingaben vor jeder Mutation und bieten bei Operationen über mehrere Assets eine Vorschau oder einen Dry Run. Lass den MCP-Server an Loopback gebunden. Stelle ihn nicht öffentlich bereit, proxye oder tunnle ihn nicht, richte kein Port-Forwarding ein und veröffentliche ihn nicht per Firewall-Regel; der lokale Endpunkt bietet keine Authentifizierung.

## Sichere SubGraph-Bearbeitung

Verwende `MayDialogueMCP.MayDialogueGraphToolset.ConfigureSubgraph`, nachdem du über `InspectDialogueAsset` die exakte `nodeGuid` eines SubGraph-Source-Nodes und die aktuelle Asset-`revision` erhalten hast. Das Tool konfiguriert Anzeigename und Rückkehrverhalten und kann den eingebetteten Graph bei Bedarf anlegen:

```json
{
  "assetPath": "/Game/Dialogues/DA_QuestGuide",
  "expectedRevision": "A1B2C3D4",
  "nodeGuid": "11111111-2222-3333-4444-555555555555",
  "subGraphName": "Quest Accepted",
  "bReturnAfterSubGraph": true,
  "bCreateIfMissing": true,
  "bSave": false
}
```

Wenn die Erstellung erlaubt ist, beginnt ein fehlender Graph als gültiges **Entry → Say Line → Exit**-Gerüst. Das Ergebnis liefert `graphPath`, `entryNodeGuid`, `exitNodeGuids`, `nodeCount`, Vorher-/Nachher-Revisionen sowie `bCreated`/`bChanged`. Inspiziere das Asset vor der nächsten Mutation erneut und verbinde den übergeordneten SubGraph-Node mit `ConnectPins` als getrennte, explizite Operation.

Setze `bCreateIfMissing=false`, wenn du ausschließlich einen vorhandenen eingebetteten Graph aktualisieren willst. Fehlt der Graph, liefert das Tool dann `MayDialogue.Subgraph.NotConfigured`, ohne den Source-Zustand zu ändern. Fremde oder nicht registrierte Graph-Referenzen werden abgelehnt; das Tool löscht, trennt, importiert, verschiebt oder errät keinen Graph. Speichern bleibt explizit und ist standardmäßig deaktiviert.

## Einstieg

1. Folge [Client-Einrichtung](client-setup.md), um Unreals lokalen MCP-Server zu starten und deinen Client zu verbinden.
2. Lass den Client `MayDialogueDocumentation.SearchDocumentation` für dein Thema aufrufen.
3. Verwende die zurückgegebene Seiten-ID mit `ReadDocumentationPage` oder den Abschnittsanker mit `ReadDocumentationSection`.
4. Übergib bei Graph-Validator-Funden die exakte `MDV.*`-ID an `ExplainDiagnostic`.

Für den spielbaren Plugin-Workflow geht es im [Quick Start](../getting-started/quick-start.md) weiter.
