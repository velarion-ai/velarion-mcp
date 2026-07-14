# Velarion Company Intelligence — MCP Server

**Executive-compensation, say-on-pay, and governance data for US public companies, exposed to AI agents over the Model Context Protocol.**

An agent connects, self-issues a token in one HTTP call (no human in the loop), and immediately queries structured comp and governance data. Read tools and three free Governance Alpha Cards per day work on the free tier; deeper artifacts settle through a metered pay rail.

- **Endpoint:** `https://velarion-scraper-production.up.railway.app/mcp` (streamable-HTTP)
- **Registry:** `ai.velarion/company-intelligence` on the [official MCP Registry](https://registry.modelcontextprotocol.io)
- **Website:** https://intel.velarion.ai

---

## What it covers

A **2,660-company universe** — ~1,200 active tickers across 12 industries plus their disclosed peer companies. Of these, **2,412 companies have extracted executive compensation**, drawn from **37,500+ executive-comp records**, **49,800+ director-comp records**, and **18,800+ say-on-pay results**. Data is sourced from annual proxy filings and refreshed nightly; every response is current as of the most recently processed filing.

A company outside the covered universe returns `not_in_coverage` rather than a guess — the server never fabricates a figure for a company it does not track.

**12 industries:** Banking · Real Estate · Tech (Software) · Biopharma & Life Sciences · Tech (Hardware) · Insurance · MedTech & Diagnostics · Utilities · Asset Management · Gaming & Lodging · Payments & Financial Infrastructure · Specialty Finance.

---

## Tools

| Tool | What it returns | Tier |
|---|---|---|
| `lookup_company_compensation` | Full Summary Comp Table for every named executive: salary, bonus, stock/option awards, non-equity incentive, total. | Free (read) |
| `benchmark_executive_pay` | CEO pay percentile vs. the company's disclosed peer set, TSR percentile, pay-for-performance gap, alignment label. | Free (read) |
| `predict_say_on_pay_risk` | Say-on-pay approval trend, year-over-year change, risk band, governance-friction signals. | Free (read) |
| `compare_companies` | Side-by-side comp and governance across two or more covered companies. | Free (read) |
| `generate_governance_alpha_card` | A one-page governance snapshot for a company. | 3 free/day |
| `price_product` | Quote a custom artifact (peer-cohort audit, extended governance pull, say-on-pay window). | Free |
| `place_order` | Open a paid order against a quote. | Metered |
| `fulfill_paid_order` | Retrieve the artifact once settlement clears. | Metered |

Paid artifacts are withheld until payment clears, and every artifact is checked against the underlying data before it ships — the server will refuse to deliver a narrative it can't ground.

---

## Quick start

### 1. Self-issue a token (one call, instant)

```bash
curl -X POST https://velarion-scraper-production.up.railway.app/agent/v1/token/self-serve \
  -H 'Content-Type: application/json' \
  -d '{"agent_label": "my-agent", "contact_email": "you@example.com"}'
```

Returns your token once — store it:

```json
{
  "token": "vlragt_...",
  "scopes": ["mcp:read", "mcp:buy"],
  "balance_cents": 0,
  "mcp_endpoint": "https://velarion-scraper-production.up.railway.app/mcp",
  "header": "X-Velarion-Agent-Token",
  "free_tier": {
    "governance_alpha_card": "3 free cards per day",
    "read_tools": "lookup_company_compensation, benchmark_executive_pay, predict_say_on_pay_risk, compare_companies"
  }
}
```

### 2. Register the server with your client

**Claude Desktop / Claude Code** (`claude_desktop_config.json` or via `claude mcp add`):

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

### 3. Ask

Once connected, ask your agent things like:

> "What did Microsoft's CEO earn last year, and how does it compare to disclosed peers?"
> "Is JPMorgan at risk of a say-on-pay revolt?"
> "Compare CEO pay-for-performance alignment across the three largest banks in coverage."

---

## Sample outputs

These are real responses from the live server, lightly trimmed.

**`lookup_company_compensation("MSFT")`**

```json
{
  "ticker": "MSFT",
  "company_name": "Microsoft Corp",
  "fiscal_year": 2025,
  "executives": [
    {
      "last_name": "Nadella", "first_name": "Satya",
      "position": "CEO", "title": "Chairman and Chief Executive Officer",
      "sct_salary": 2500000, "sct_stock_awards": 84245496,
      "sct_non_equity_incentive": 9555000, "sct_all_other": 196294,
      "total_comp": 96496790, "is_departed": false
    },
    {
      "last_name": "Hood", "first_name": "Amy",
      "position": "CFO", "title": "Executive Vice President and Chief Financial Officer",
      "sct_salary": 1000000, "sct_stock_awards": 25037360,
      "sct_non_equity_incentive": 3421000, "total_comp": 29481551
    }
  ]
}
```

**`predict_say_on_pay_risk("JPM")`**

```json
{
  "ticker": "JPM",
  "risk_band": "low",
  "latest_sop_approval_pct": 92.74,
  "sop_yoy_change_bps": 91,
  "trend_phrase": "Say-on-Pay approval trend: FY2024 91.8% → FY2025 92.7%.",
  "governance_friction_summary": "Governance friction signals: none identified from available data.",
  "limitations": "Deterministic signal only. Does not incorporate narrative CD&A context or ISS/Glass Lewis advisory outputs."
}
```

---

## Who this is for

Comp consultants, institutional investors, corporate compensation committees, governance analysts — and the agents that work on their behalf. Velarion structures what public companies disclose about how they pay their leadership, so an agent can answer a benchmarking or governance question in one call instead of parsing a filing.

## Trust & limitations

- **Sourced from annual proxy filings**, processed nightly. A response is current as of the most recently processed filing for that company.
- **Deterministic where it matters.** Say-on-pay and benchmarking signals are computed from disclosed figures, not model guesses. Narrative outputs are grounded against the underlying numbers before delivery.
- **Coverage is bounded and honest.** Outside the ~2,660-company universe, tools return `not_in_coverage`.
- Not investment advice. Not a substitute for a company's own filings.

## Support

Questions, coverage requests, or integration help: **andy@velarion.ai**
