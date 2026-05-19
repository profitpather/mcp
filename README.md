# Profitpather — MCP server

[![MCP](https://img.shields.io/badge/MCP-streamable--http-blue)](https://modelcontextprotocol.io)
[![OAuth 2.1](https://img.shields.io/badge/auth-OAuth%202.1-green)](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

**Profitpather** is a first-party, unsampled analytics layer for Shopify stores. This is its public **Model Context Protocol** server — 26 tools that give AI agents live, raw, every-event access to merchant traffic, attribution, funnels, and revenue-leak diagnostics.

- **Registry name:** `io.github.profitpather/profitpather` (or `com.profitpather/profitpather` via DNS-verified namespace — see [PUBLISHING.md](./PUBLISHING.md))
- **Endpoint:** `https://profitpather.com/mcp`
- **Transport:** Streamable HTTP (protocol versions `2025-11-25` · `2025-06-18` · `2025-03-26` · `2024-11-05`)
- **Auth:** OAuth 2.1 with Dynamic Client Registration (RFC 7591)
- **Docs (LLM-ready):** [profitpather.com/llms-full.txt](https://profitpather.com/llms-full.txt)
- **Server card:** [profitpather.com/.well-known/mcp/server-card.json](https://profitpather.com/.well-known/mcp/server-card.json)

---

## Why Profitpather

| Alternative | Limitation Profitpather solves |
|---|---|
| GA4 / Shopify Reports | Aggressively sampled, third-party-cookie dependent, drops UTMs across the Shopify cross-domain checkout — designed to feed Google Ads, not to answer "where did this cart actually start?" |
| Meta / TikTok / Google Ads dashboards | Self-reported clicks, regularly over-attribute, can't see on-domain behaviour |
| Triple Whale / Northbeam / Polar ($150–$1,500/mo) | Closed paid suites, not Claude-native, not MCP-accessible |
| Heap / Mixpanel / Amplitude | Generic product analytics — don't understand Shopify cart-goal logic, pre-cart paths, checkout linkage, store-domain attribution |

Profitpather is **unsampled**, **first-party**, **Shopify-native**, and **agent-native by design** — analyse in chat, ship fixes from chat.

---

## Tools (26)

Headline tools (the ones agents reach for first):

| Tool | Purpose |
|---|---|
| `pp_list_domains` | List every Shopify domain this account can query. Call first. |
| `pp_brief` | One-call first look — overview + acquisition + device + pixel-status in parallel, with a pre-formatted markdown presentation. |
| `pp_diagnose_leaks` | Revenue-leak finder. Aggregates 8 analyses into a prioritised `recommendations[]` array. |
| `pp_overview` | Top-level KPIs for one window: sessions, cart rate, checkout rate, top sources, top landing pages. |
| `pp_acquisition` | Full attribution matrix — every (source, medium, campaign, utm, click-id) combo with revenue, AOV, conversion rates. |
| `pp_conversion` | Funnel + pre-cart paths + checkout economics broken down by stage. |
| `pp_path_analysis` | Page-to-page flows with cart-rate weighting; finds dead-end transitions. |
| `pp_markov` | Markov-chain attribution model across multi-touch journeys. |
| `pp_in_app_browser_loss` | Revenue at risk from in-app browsers (Instagram, TikTok, Facebook). |
| `pp_session_trace` | Full per-session event timeline for a single visitor. |
| `pp_pixel_status` | Pixel health & coverage check. |
| `pp_compare_periods` | Period-over-period comparison with statistical significance. |
| `pp_ai_insights` | LLM-summarised insights pre-computed nightly. |
| `pp_signals` | All measured behavioural signals on a domain. |

Plus 12 more covering device splits, page-lift attribution, product analytics, search, timeseries, real-time today-view, change log, admin status, etc.

Full per-tool schemas at: [profitpather.com/.well-known/agent-skills/index.json](https://profitpather.com/.well-known/agent-skills/index.json)

---

## Install

### Claude Desktop / Claude Code / Cursor

```json
{
  "mcpServers": {
    "profitpather": {
      "url": "https://profitpather.com/mcp"
    }
  }
}
```

Restart the client. On first tool call, you'll be redirected through OAuth 2.1 to authorise the connection. Sign in with your Profitpather account, approve, and the client stores the token automatically.

### ChatGPT / Claude.ai Custom Connectors

Settings → Connectors → Add custom connector → paste `https://profitpather.com/mcp` → approve OAuth.

### No Profitpather account yet?

Sign up at [profitpather.com](https://profitpather.com), install the single Shopify Custom Pixel on your store, and Profitpather starts collecting events immediately. The MCP becomes useful within minutes of the pixel firing.

---

## Architecture

Profitpather runs entirely on **Cloudflare Workers** with **D1** (SQLite at the edge) for per-merchant event storage. The MCP endpoint at `/mcp` is the same Worker that serves the dashboard at `profitpather.com/dashboard` — analytics queries hit D1 directly with no intermediate cache or sampling.

```
Shopify Custom Pixel (storefront)
       │
       ▼
Cloudflare Worker  ───────►  D1 (per-merchant event store)
       │                              ▲
       ▼                              │
   /mcp (MCP server)  ─── reads ──────┘
       │
       ▼
  AI agent (Claude / ChatGPT / Cursor / your app)
```

All 26 MCP tools are read-only against the merchant's own data — no cross-tenant access, no third-party data sharing.

---

## Discovery endpoints

Profitpather publishes the full agentic-discovery surface:

| Path | What it returns |
|---|---|
| `/.well-known/mcp/server-card.json` | MCP server card v1 (SEP-1649) |
| `/.well-known/oauth-protected-resource` | OAuth 2.1 discovery (RFC 9728) |
| `/.well-known/api-catalog` | RFC 9727 API catalogue |
| `/.well-known/agent-skills/index.json` | Per-tool agent skills with SHA-256 digests (Cloudflare RFC v0.2.0) |
| `/llms-full.txt` | Full Markdown documentation for LLM ingestion |
| `/llms.txt` | LLM index |

---

## License

MIT — see [LICENSE](LICENSE).

Built by [Boolsai](https://boolsai.ai) · Contact: [founder@boolsai.ai](mailto:founder@boolsai.ai)
