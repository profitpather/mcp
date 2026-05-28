# CLAUDE.md

Guidance for AI assistants working in this repository.

## What this repository is

This is the **public registry-manifest and documentation repo** for the
Profitpather MCP server. It is **not** the server's source code.

- The running MCP server lives on **Cloudflare Workers** at
  `https://profitpather.com/mcp` (Worker + D1, in a separate private repo).
- This repo exists only to (a) hold the `server.json` manifest that is
  published to the **Anthropic / Model Context Protocol registry**, and
  (b) provide the human-facing `README.md` that GitHub and registry
  aggregators link to.

So there is **no build, no test suite, no runtime code** here. Changes are
edits to Markdown/JSON, validated and published with the `mcp-publisher` CLI.

## Files

| File | Purpose |
|---|---|
| `server.json` | The registry manifest. The single source of truth for the published registry entry. Follows the MCP server schema declared in its `$schema` field. |
| `README.md` | Human-facing overview: positioning, the 26-tool list, install snippets, architecture, discovery endpoints. |
| `PUBLISHING.md` | Operator runbook for publishing `server.json` to the registry (DNS auth, login, publish, deprecate). |
| `LICENSE` | MIT. |

## Core workflow: publishing a manifest update

`server.json` is the only "deployable" artifact. To change the registry entry:

1. Edit `server.json`.
2. **Bump the `version` field (semver).** The registry rejects re-publishing
   the same version — this is the most common mistake.
3. Validate with no side effects: `mcp-publisher validate`
   (expects `✅ server.json is valid`).
4. Publish: `mcp-publisher publish` (reads `./server.json` by default).

See `PUBLISHING.md` for DNS-challenge login (`mcp-publisher login dns …`),
key handling, and deprecating old versions. The actual publish requires the
Ed25519 private key at `~/.config/mcp-publisher/profitpather.key.pem`, which
is not in this repo — **do not attempt to publish without the operator's
explicit go-ahead**, and never commit keys.

## Conventions / invariants

- **`name` is permanent.** `com.profitpather/analytics` is locked for the life
  of the registry entry. Every other field is mutable per-version. Never rename
  it. (Git history shows an earlier rename from `com.profitpather/profitpather`
  before anything was locked in — that window is closed.)
- **DNS-verified namespace.** The `com.profitpather/*` namespace is authorized
  via a TXT record at the `profitpather.com` apex. Authority derives from
  controlling that domain.
- **Keep README and `server.json` consistent.** If the tool count, endpoint,
  description, or version changes, update both. The README's "Tools (26)" and
  the manifest `description` should not drift apart.
- **Endpoint and transport are fixed contracts.** `https://profitpather.com/mcp`
  over Streamable HTTP with OAuth 2.1. Don't change these in docs unless the
  live server actually changed.
- The canonical published entry is `com.profitpather/analytics` — there is one
  manifest. A prior `profitpather`/`analytics` dual-manifest setup was
  intentionally collapsed to a single canonical file; don't reintroduce variants.

## Git workflow

- Develop on the designated feature branch; commit with clear messages.
- Push with `git push -u origin <branch>`.
- Do **not** open a pull request unless explicitly asked.

## Pointers to the real system (for context, not editable here)

The server's behavior, data model, and tools are documented at the live
discovery endpoints rather than in this repo:

- `https://profitpather.com/llms-full.txt` — full LLM-ready docs
- `https://profitpather.com/.well-known/mcp/server-card.json` — server card
- `https://profitpather.com/.well-known/agent-skills/index.json` — per-tool schemas

Data model in one line: every visitor event is one row in an `events` table
(`ts`, `session_id`, `signal`, `path`, `payload` JSON) in per-merchant D1; all
26 MCP tools are read-only against the merchant's own data.
