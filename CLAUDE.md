# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is the **public registry manifest and documentation** for the Profitpather MCP server — it contains no application code. The actual MCP server is a Cloudflare Worker (with D1 storage) deployed at `https://profitpather.com/mcp`; that code lives elsewhere. This repo exists so the server can be published to the Anthropic MCP Registry under the DNS-verified name `com.profitpather/analytics` and so the GitHub repo linked from the registry has meaningful docs.

Files:

- `server.json` — the MCP registry manifest (the only "source" file). Schema: `https://static.modelcontextprotocol.io/schemas/2025-12-11/server.schema.json`.
- `README.md` — public-facing docs: tool list (26 tools), install instructions, architecture, discovery endpoints.
- `PUBLISHING.md` — the registry publishing runbook (DNS auth, `mcp-publisher` commands).

## Commands

There is no build, lint, or test step. The only tooling is `mcp-publisher` (run locally by the maintainer, not in CI):

```bash
mcp-publisher validate    # validate server.json — no publish, no side effects
mcp-publisher publish     # publish server.json to the MCP registry (requires DNS-key login, see PUBLISHING.md)
```

## Key conventions

- **Any change to `server.json` that will be republished must bump the `version` field** (semver) — the registry rejects re-publishing an existing version. Current published version is tracked in both `server.json` and the "Live at" line of `PUBLISHING.md`; keep them in sync.
- The `name` field `com.profitpather/analytics` is **permanent** — never change it. Every other field in `server.json` is mutable per-version.
- Keep `README.md` consistent with the live server: the tool count (currently 26), the endpoint URL, supported protocol versions, and the discovery-endpoint table all describe the deployed Worker. Don't invent or rename tools — the authoritative per-tool schemas are at `profitpather.com/.well-known/agent-skills/index.json`.
- The three docs cross-reference the same facts (registry name, endpoint, version). When editing one, check whether the others need the same update.
