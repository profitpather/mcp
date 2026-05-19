# Publishing to the Anthropic MCP Registry

This repo contains **two ready-to-publish manifests** for the same Profitpather MCP server. Pick which namespace to ship under.

## The two options

| File | Namespace | Auth required | Trust signal |
|---|---|---|---|
| [`server.json`](./server.json) | `io.github.profitpather/profitpather` | GitHub OAuth (`mcp-publisher login github`) — already done | "Published by the `profitpather` GitHub account" |
| [`server.com.json`](./server.com.json) | `com.profitpather/profitpather` | DNS TXT challenge on `profitpather.com` (Ed25519 keypair) | "Published by the entity that controls profitpather.com" (verified domain — stronger signal) |

Both point at the **same live endpoint** (`https://profitpather.com/mcp`) and expose the **same 26 tools**. The only difference is the registry name and the trust signal in aggregator listings (Smithery / PulseMCP / MCP.so / GitHub MCP Registry).

> **You can publish both** — they live as separate registry entries. Some teams ship the GitHub-namespace one first (fast), then add the DNS-verified one once they've validated the workflow. Recommend doing **either**, not both, to avoid splitting install counts.

---

## Option A — publish `io.github.profitpather/profitpather` (fast path, no DNS work)

```bash
mcp-publisher login github   # OAuth device-code flow, one-time
mcp-publisher publish        # uses server.json by default
```

Within minutes the entry appears at `https://registry.modelcontextprotocol.io/v0.1/servers?search=profitpather`.

---

## Option B — publish `com.profitpather/profitpather` (DNS-verified path, ~5 min DNS dance)

1. **Generate Ed25519 keypair locally:**
   ```bash
   openssl genpkey -algorithm Ed25519 -out key.pem
   PUBLIC_KEY=$(openssl pkey -in key.pem -pubout -outform DER | tail -c 32 | base64)
   echo "Add this TXT record to profitpather.com DNS:"
   echo "profitpather.com. IN TXT \"v=MCPv1; k=ed25519; p=${PUBLIC_KEY}\""
   ```

2. **Add the printed TXT record** to your DNS provider for `profitpather.com` at the apex (`@`).

3. **Wait ~30 seconds for DNS propagation, then verify:**
   ```bash
   dig +short TXT profitpather.com | grep MCPv1
   ```

4. **Authenticate via DNS challenge:**
   ```bash
   PRIVATE_KEY=$(openssl pkey -in key.pem -noout -text | grep -A3 'priv:' | tail -n +2 | tr -d ' :\n')
   mcp-publisher login dns --domain profitpather.com --private-key "$PRIVATE_KEY"
   ```

5. **Publish using the `com.*` manifest:**
   ```bash
   mcp-publisher publish -f server.com.json
   ```

6. **(Optional) Once verified once, you can drop the TXT record** — DNS verification is needed only at publish time, not for ongoing operation. Consider keeping it for re-publish convenience though.

---

## Updating after publish

Bump the `version` field in whichever `server.json` is live (e.g. `0.3.7` → `0.3.8`) and re-run `mcp-publisher publish`. All other fields are mutable per-version (description, URL, repository, etc.). The `name` is permanent.

To deprecate an old version:

```bash
mcp-publisher status --version 0.3.7 --status deprecated
```

To migrate from the `io.github` namespace to `com.profitpather` later: publish `com.profitpather/profitpather` fresh, then deprecate the `io.github.profitpather/profitpather` versions. Install counts don't transfer between names.
