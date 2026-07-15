# Custom requests — pricing and fulfilling artifacts beyond the read tools

The read tools (`lookup_company_compensation`, `benchmark_executive_pay`,
`predict_say_on_pay_risk`, `compare_companies`) and the 3-free/day Governance Alpha Card cover
most agent questions for free. When an agent needs a **deeper, packaged artifact** — a peer-cohort
audit, an extended governance pull, a multi-year say-on-pay window — that runs through the paid
rail: `list_skus` → `price_product` → `place_order` → `fulfill_paid_order`.

There is no separate "custom request" endpoint. Bespoke asks are modeled as **custom artifact
families** in the same catalog, so the same four tools handle both fixed SKUs and custom work.

## 1. Discover what's sellable

```
list_skus()
```

Returns every sellable item — fixed SKUs (`SKU-002`, `SKU-003`, `SKU-004`, …) and custom artifact
families (e.g. `peer_disclosure_custom_cohort`) — each with its pricing shape (`one_off_fixed`,
`free_tier`, or a profit-aware custom quote) and expected latency.

## 2. Get a live, profit-aware quote

```
price_product(product_type="peer_disclosure_custom_cohort", tickers=["MSFT","CRM","NOW"])
```

`product_type` accepts either a `sku_id` or a custom artifact family. The quote is computed live
and **respects the configured margin floor** — the server never returns a price that undercuts it.
Out-of-coverage tickers are surfaced separately and excluded from the priced set.

## 3. Open the order, then settle

```
place_order(quote_id="...")
```

Opens a paid order against the quote. Fund the order's balance via USDC on Base (or a marketplace
entitlement that maps to a funded token).

## 4. Retrieve the artifact

```
fulfill_paid_order(order_id="...")
```

Fail-closed gate: the artifact is compiled and returned **only** after settlement is verified, and
every artifact is grounded against the underlying data before it ships — the server refuses to
deliver a narrative it cannot support.

## Something genuinely outside the catalog?

If you need an artifact that no SKU or custom family covers — a new report type, a bulk cohort, an
industry-wide cut — email **andy@velarion.ai** with the companies, the question, and the format you
need. Net-new artifact types are scoped and priced by a human before they enter the catalog; once
added, they're quotable through `price_product` like everything else.
