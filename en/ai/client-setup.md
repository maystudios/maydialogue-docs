---
description: "Connect Codex, Claude Code, or OpenCode to MayDialogue's local Unreal Engine MCP endpoint."
---

# MCP Client Setup

MayDialogue's MCP tools are served by Unreal Engine 5.8's experimental Model Context Protocol plugin. The default local endpoint is:

```text
http://127.0.0.1:8000/mcp
```

No API key, token, or MayDialogue credential belongs in these examples. The endpoint is a local editor service; your AI client handles its own model credentials separately.

{% hint style="warning" %}
Keep this endpoint on `127.0.0.1`. Do not expose, proxy, tunnel, port-forward, or firewall-publish it. Loopback access is not an authentication mechanism, and the Unreal MCP server does not add authentication for you.
{% endhint %}

## Prepare Unreal Engine

1. In **Edit → Plugins**, enable **Model Context Protocol**. MayDialogue enables its required Toolset Registry dependency.
2. Restart the editor when prompted.
3. In **Project Settings → Plugins → Model Context Protocol**, enable **Auto Start Server**, or run `ModelContextProtocol.StartServer` in the Unreal console for the current editor session.
4. Keep the default port `8000` and path `/mcp` unless another local service already uses that port.
5. Leave the Unreal Editor open while the client calls MayDialogue tools.

The integration is available only in editor targets. A packaged game does not expose this endpoint.

## Codex

Register the local server from a terminal:

```powershell
codex mcp add maydialogue --url http://127.0.0.1:8000/mcp
```

Or add the following to your Codex `config.toml`:

```toml
[mcp_servers.maydialogue]
url = "http://127.0.0.1:8000/mcp"
```

Restart Codex after changing the configuration, then ask it to list MayDialogue tools or call `MayDialogueDocumentation.GetQuickStart`.

## Claude Code

Register the streamable HTTP server in the current project:

```powershell
claude mcp add --transport http maydialogue http://127.0.0.1:8000/mcp
```

Claude Code also supports local, project, and user scopes. Use the narrowest scope that matches your workflow; a project-scoped `.mcp.json` can be shared with collaborators when it contains only this loopback URL and no secrets.

Restart the client session after changing the configuration, then use `/mcp` to inspect the connection.

## OpenCode

Add a remote MCP server entry to `opencode.json`:

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

Restart OpenCode after saving the file, then ask it to list the tools supplied by `maydialogue`.

## Verify the connection

With Unreal Editor open and the server running, call:

```text
MayDialogueDocumentation.GetClientSetup(client="codex", language="en")
```

A successful result contains the same documentation bundle identity as every other documentation response. If the connection fails, confirm that the editor is running, the server was started, the URL matches Project Settings, and no firewall or other process blocks the selected local port.

Client configuration syntax can change independently of MayDialogue. The canonical upstream references are the [Codex MCP documentation](https://developers.openai.com/codex/codex-manual.md), [Claude Code MCP documentation](https://docs.anthropic.com/en/docs/claude-code/mcp), and [OpenCode MCP servers documentation](https://opencode.ai/docs/mcp-servers/).
