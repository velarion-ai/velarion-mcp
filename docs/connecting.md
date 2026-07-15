# Connecting an agent to Velarion Company Intelligence

This is the full connection flow for any MCP client — Claude Desktop, Claude Code, a custom
agent, or an agent procured through a marketplace such as AWS Marketplace. The server is a
remote streamable-HTTP MCP endpoint; there is nothing to install.

- **Endpoint:** `https://velarion-scraper-production.up.railway.app/mcp`
- **Transport:** streamable-HTTP
- **Auth header:** `X-Velarion-Agent-Token`

## 1. Get a token

Every agent authenticates with a per-agent token. Self-issue one in a single call — no human
in the loop, no dashboard:

```bash
curl -X POST https://velarion-scraper-production.up.railway.app/agent/v1/token/self-serve \
  -H 'Content-Type: application/json' \
  -d '{"agent_label": "my-agent", "contact_email": "you@example.com"}'
```

The raw token is returned **once** — store it. Read tools and three free Governance Alpha Cards
per day work immediately; paid products require a funded balance.

> **Procured Velarion through a marketplace?** Your marketplace entitlement maps to a funded
> token — the marketplace registration flow provisions it for you, so paid tools work without a
> separate top-up. The self-serve flow above is the free-tier path for direct connections.

## 2. Register the endpoint with your client

### Claude Desktop / Claude Code

```json
{
  "mcpServers": {
    "velarion": {
      "type": "streamable-http",
      "url": "https://velarion-scraper-production.up.railway.app/mcp",
      "headers": {
        "X-Velarion-Agent-Token": "vlragt_your_token_here"
      }
    }
  }
}
```

The same config as a file is committed at [`examples/claude_desktop_config.json`](../examples/claude_desktop_config.json).

### Cursor

Add to `~/.cursor/mcp.json` (or a project `.cursor/mcp.json`). Cursor infers a remote server from the `url`:

```json
{
  "mcpServers": {
    "velarion": {
      "url": "https://velarion-scraper-production.up.railway.app/mcp",
      "headers": { "X-Velarion-Agent-Token": "vlragt_your_token_here" }
    }
  }
}
```

Committed at [`examples/cursor_mcp_config.json`](../examples/cursor_mcp_config.json).

### VS Code

VS Code's native MCP support uses a `servers` block (`.vscode/mcp.json` or user settings):

```json
{
  "servers": {
    "velarion": {
      "type": "http",
      "url": "https://velarion-scraper-production.up.railway.app/mcp",
      "headers": { "X-Velarion-Agent-Token": "vlragt_your_token_here" }
    }
  }
}
```

Committed at [`examples/vscode_mcp_config.json`](../examples/vscode_mcp_config.json).

### A custom agent (raw MCP over HTTP)

Standard MCP handshake — `initialize`, capture the `mcp-session-id` response header, then
`tools/list` and `tools/call`, sending the token header on every request:

```bash
BASE=https://velarion-scraper-production.up.railway.app
# initialize (capture the mcp-session-id response header)
curl -si -X POST $BASE/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -H "X-Velarion-Agent-Token: $TOKEN" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize",
       "params":{"protocolVersion":"2024-11-05","capabilities":{},
                 "clientInfo":{"name":"my-agent","version":"1.0"}}}'

# then call a tool, echoing the session id back
curl -s -X POST $BASE/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -H "X-Velarion-Agent-Token: $TOKEN" \
  -H "mcp-session-id: $SESSION_ID" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call",
       "params":{"name":"lookup_company_compensation","arguments":{"ticker":"MSFT"}}}'
```

## 3. Tools

| Tool | Returns | Tier |
|---|---|---|
| `lookup_company_compensation` | Full Summary Comp Table per named executive | Free (read) |
| `benchmark_executive_pay` | CEO pay percentile vs disclosed peers, TSR, alignment | Free (read) |
| `predict_say_on_pay_risk` | Say-on-pay trend, risk band, friction signals | Free (read) |
| `compare_companies` | Side-by-side comp/governance across companies | Free (read) |
| `generate_governance_alpha_card` | One-page governance snapshot | 3 free/day |
| `price_product` | Quote a custom artifact | Free |
| `place_order` | Open a paid order against a quote | Metered |
| `fulfill_paid_order` | Retrieve the artifact once settlement clears | Metered |
| `list_skus` | Enumerate the sellable catalog (SKUs + custom families) | Free |

Paid artifacts are withheld until payment clears, and every artifact is grounded against the
underlying data before delivery — the server refuses to ship a narrative it cannot support.

## 4. What a call looks like

`lookup_company_compensation("MSFT")` returns, per executive: `sct_salary`, `sct_bonus`,
`sct_stock_awards`, `sct_option_awards`, `sct_non_equity_incentive`, `sct_all_other`,
`total_comp`, plus `fiscal_year` and `company_name`. Outside the ~2,660-company coverage
universe, tools return `not_in_coverage` rather than a guess.

## Support

`andy@velarion.ai`
