---
description: "Codex, Claude Code oder OpenCode mit MayDialogues lokalem Unreal-Engine-MCP-Endpunkt verbinden."
---

# MCP-Client-Einrichtung

MayDialogues MCP-Tools werden vom experimentellen Model-Context-Protocol-Plugin von Unreal Engine 5.8 bereitgestellt. Der lokale Standard-Endpunkt lautet:

```text
http://127.0.0.1:8000/mcp
```

In diese Beispiele gehören weder API-Key noch Token oder MayDialogue-Zugangsdaten. Der Endpunkt ist ein lokaler Editor-Dienst; Zugangsdaten für das verwendete Modell verwaltet dein KI-Client getrennt.

## Unreal Engine vorbereiten

1. Aktiviere unter **Edit → Plugins** das Plugin **Model Context Protocol**. MayDialogue aktiviert dessen benötigte Toolset-Registry-Abhängigkeit.
2. Starte den Editor neu, wenn du dazu aufgefordert wirst.
3. Aktiviere unter **Project Settings → Plugins → Model Context Protocol** die Option **Auto Start Server**, oder führe für die aktuelle Editor-Sitzung `ModelContextProtocol.StartServer` in der Unreal-Konsole aus.
4. Behalte Port `8000` und Pfad `/mcp` bei, sofern kein anderer lokaler Dienst diesen Port verwendet.
5. Lass den Unreal Editor geöffnet, während der Client MayDialogue-Tools aufruft.

Die Integration steht nur in Editor-Targets zur Verfügung. Ein paketiertes Spiel stellt diesen Endpunkt nicht bereit.

## Codex

Registriere den lokalen Server in einem Terminal:

```powershell
codex mcp add maydialogue --url http://127.0.0.1:8000/mcp
```

Alternativ ergänzt du die Codex-`config.toml`:

```toml
[mcp_servers.maydialogue]
url = "http://127.0.0.1:8000/mcp"
```

Starte Codex nach der Konfigurationsänderung neu. Lass anschließend die MayDialogue-Tools auflisten oder `MayDialogueDocumentation.GetQuickStart` aufrufen.

## Claude Code

Registriere den Streamable-HTTP-Server im aktuellen Projekt:

```powershell
claude mcp add --transport http maydialogue http://127.0.0.1:8000/mcp
```

Claude Code unterstützt außerdem lokale, projektweite und benutzerweite Scopes. Verwende den engsten passenden Scope. Eine projektweite `.mcp.json` kann mit dem Team geteilt werden, wenn sie ausschließlich diese Loopback-URL und keine Geheimnisse enthält.

Starte die Client-Sitzung nach der Änderung neu und prüfe die Verbindung mit `/mcp`.

## OpenCode

Ergänze in `opencode.json` einen Remote-MCP-Server:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "maydialogue": {
      "type": "remote",
      "url": "http://127.0.0.1:8000/mcp",
      "enabled": true
    }
  }
}
```

Starte OpenCode nach dem Speichern neu und lass die vom Server `maydialogue` bereitgestellten Tools auflisten.

## Verbindung prüfen

Rufe bei geöffnetem Unreal Editor und laufendem Server Folgendes auf:

```text
MayDialogueDocumentation.GetClientSetup(client="codex", language="de")
```

Ein erfolgreiches Ergebnis enthält dieselbe Dokumentations-Bundle-Identität wie jede andere Dokumentationsantwort. Schlägt die Verbindung fehl, prüfe, ob der Editor läuft, der Server gestartet wurde, die URL mit den Project Settings übereinstimmt und weder Firewall noch anderer Prozess den lokalen Port blockieren.

Die Syntax der Client-Konfiguration kann sich unabhängig von MayDialogue ändern. Maßgeblich sind die Upstream-Referenzen für [Codex MCP](https://developers.openai.com/codex/codex-manual.md), [Claude Code MCP](https://docs.anthropic.com/en/docs/claude-code/mcp) und [OpenCode MCP-Server](https://opencode.ai/docs/mcp-servers/).
