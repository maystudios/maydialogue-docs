---
description: "Use MayDialogue's editor-only MCP tools and shipped documentation with an external AI client."
---

# AI Clients and MCP

MayDialogue ships an editor-only Model Context Protocol (MCP) integration for Unreal Engine 5.8. It lets an MCP-capable client inspect and author dialogue assets, Babel profiles, localization data, and the versioned MayDialogue documentation bundle through typed tools.

{% hint style="info" %}
**MayDialogue does not contain, train, or run an AI model.** Your external client supplies its own model and combines that model with the documentation and tools exposed by the running Unreal Editor.
{% endhint %}

The MCP integration is deliberately editor-only. It is not loaded or packaged in Shipping builds, and it does not add an AI runtime dependency to your game.

## What the documentation tools expose

The shipped bundle is generated from the same bilingual pages that power this GitBook. Every tool result identifies the plugin version, documentation commit, bundle content hash, language, and page or section it came from.

An MCP client can:

* search the English or German documentation;
* read a complete page or one named section;
* retrieve the canonical five-minute Quick Start;
* explain a stable `MDV.*` validator diagnostic;
* retrieve current setup instructions for Codex, Claude Code, or OpenCode.

The tools return real shipped documentation, not a separate collection of marketing answers. A deterministic drift check fails during development if the generated bundle no longer matches the canonical docs.

## Safety model

Documentation tools are read-only. Authoring tools use structured requests, validate inputs before mutation, and expose preview or dry-run behavior where the operation can change multiple assets. Keep the MCP server bound to loopback unless you deliberately secure a different network arrangement.

## Start here

1. Follow [Client Setup](client-setup.md) to start Unreal's local MCP server and connect your client.
2. Ask the client to call `MayDialogueDocumentation.SearchDocumentation` for a topic.
3. Use the returned page ID with `ReadDocumentationPage` or the returned section anchor with `ReadDocumentationSection`.
4. For graph validation findings, pass the exact `MDV.*` ID to `ExplainDiagnostic`.

For the playable plugin workflow, continue with the [Quick Start](../getting-started/quick-start.md).
