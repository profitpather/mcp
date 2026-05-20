# Publishing to the Anthropic MCP Registry

Profitpather publishes under the **DNS-verified `com.profitpather/*` namespace** — the strongest authority signal in the registry (the entity controls `profitpather.com`).

Live at: [`com.profitpather/analytics` v0.3.7](https://registry.modelcontextprotocol.io/v0.1/servers?search=com.profitpather)

---

## One-time DNS authentication

Already set up. Private key stored at `~/.config/mcp-publisher/profitpather.key.pem`. TXT record live at the profitpather.com apex.

If you ever need to re-mint:

```bash
# 1. Generate Ed25519 keypair (use Homebrew openssl on macOS — LibreSSL doesn't support Ed25519)
/opt/homebrew/bin/openssl genpkey -algorithm Ed25519 -out ~/.config/mcp-publisher/profitpather.key.pem
chmod 600 ~/.config/mcp-publisher/profitpather.key.pem

# 2. Derive the TXT record value
PUBLIC_KEY=$(/opt/homebrew/bin/openssl pkey -in ~/.config/mcp-publisher/profitpather.key.pem -pubout -outform DER | tail -c 32 | base64)
echo "v=MCPv1; k=ed25519; p=${PUBLIC_KEY}"

# 3. Add at profitpather.com apex DNS (Cloudflare → DNS → Add record → TXT)
# 4. Verify propagation
dig +short TXT profitpather.com | grep MCPv1
```

---

## Authenticating + publishing

```bash
# Derive private key from PEM
PRIVATE_KEY=$(/opt/homebrew/bin/openssl pkey -in ~/.config/mcp-publisher/profitpather.key.pem -noout -text | grep -A3 'priv:' | tail -n +2 | tr -d ' :\n')

# Log in via DNS challenge
mcp-publisher login dns --domain profitpather.com --private-key "$PRIVATE_KEY"

# Publish (uses ./server.json by default)
mcp-publisher publish
```

---

## Updating after publish

1. Edit `server.json` (description, URL, version, etc.)
2. **Bump the `version` field** (semver — registry rejects re-publishing the same version)
3. `mcp-publisher publish`

All fields are mutable per-version EXCEPT `name` (`com.profitpather/analytics` is permanent).

---

## Validation (no publish, no side effects)

```bash
mcp-publisher validate
```

Returns `✅ server.json is valid` on success.

---

## Deprecating an old version

```bash
mcp-publisher status com.profitpather/analytics --version 0.3.7 --status deprecated
```

Aggregators (Smithery / PulseMCP / MCP.so / GitHub MCP Registry) stop surfacing deprecated versions by default.
