# Deepmerge for Claude Code

One-command install of the [Deepmerge](https://deepmerge.ai) MCP server: the shared workspace where your AI agents collaborate. This directory is a self-contained Claude Code plugin that also acts as its own marketplace.

## Install (for users)

```
/plugin marketplace add <OWNER>/<REPO>
/plugin install deepmerge@deepmerge
```

Then run `/mcp` and approve the browser sign-in once (OAuth 2.1, no API key). After that, the `get_self` / `get_context` / `create_entry` tools and the `bootstrap` / `capture` / `retrieve` slash commands are available, and the server's instructions drive the read-before / write-after loop.

Replace `<OWNER>/<REPO>` with the public GitHub repo this plugin is published to (see below).

## Update (for users)

New versions ship when the `version` in `.claude-plugin/plugin.json` is bumped. To pull the latest, refresh the marketplace, then update the plugin:

```
/plugin marketplace update deepmerge
/plugin update deepmerge@deepmerge
```

The `/plugin` menu also surfaces available updates. Updating pulls the latest skill and server config; the OAuth connection is preserved, so there is no second sign-in.

## What it does

`.mcp.json` wires Claude Code to the remote Deepmerge MCP server at `https://app.deepmerge.ai/mcp`. Installing the plugin registers the connection; the first tool call triggers the one-time OAuth sign-in.

Two layers ship behavior, and they do not overlap:

- **The server** owns the *how*: the manual at `deepmerge://workspace/AGENTS.md`, the server `instructions`, the tool schemas, and the three MCP prompts. This stays the single source of truth, so it never drifts.
- **The bundled skill** (`skills/deepmerge-workspace/SKILL.md`) owns the *when*: a trigger-dense description that fires on the cues where an agent should reach for the workspace (starting a task, producing a deliverable, resuming a session, "what did we do"), and drives the read-before / write-after loop. It points at the live tools rather than restating them, so it carries no API detail that could go stale.

While the skill is active, its `allowed-tools` auto-approves the Deepmerge MCP tools (`mcp__*deepmerge*__*`), so the read-before / write-after loop runs without a permission prompt on every call. The grant is scoped to this server and to the moments the skill triggers; nothing else is auto-approved.

The skill is pure markdown, with no scripts, no Python, and no dependencies. It is auto-discovered from the `skills/` directory; no `plugin.json` change is needed to load it.

## Publishing (for the Deepmerge team)

1. Push the **contents of this directory** to a **public** GitHub repo (e.g. `deepmerge-ai/deepmerge` or `the-s-anton/deepmerge-plugin`). The repo root must contain `.claude-plugin/` and `.mcp.json`.
2. Users then install with `/plugin marketplace add <owner>/<repo>` → `/plugin install deepmerge@deepmerge`.
3. Bump `version` in `.claude-plugin/plugin.json` on changes.

## Verify before publishing

Test locally against this directory first, no GitHub round-trip needed:

```
/plugin marketplace add /absolute/path/to/claude-plugin
/plugin install deepmerge@deepmerge
/mcp        # approve the sign-in, confirm "deepmerge ✓ connected"
```

If `/plugin` or `.mcp.json` behavior differs in your Claude Code version, check the current docs: code.claude.com/docs/en/plugins, .../plugin-marketplaces, .../mcp.
