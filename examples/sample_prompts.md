# Sample prompts

Once the Velarion MCP server is connected, these are the shapes of question it answers well. Each maps to one or two tool calls — you don't need to name the tool, the agent will pick it.

## Benchmarking a single executive

> What did Microsoft's CEO earn in the most recent fiscal year? Break out salary, equity, and incentive pay.

> How does Amy Hood's total compensation compare to CFOs at Microsoft's disclosed peer companies?

> Show me CEO pay percentile versus disclosed peers for Salesforce, and tell me whether pay and performance are aligned.

## Say-on-pay and governance risk

> Is JPMorgan at risk of a say-on-pay revolt? What's the trend?

> Which of these banks has the weakest say-on-pay trajectory: JPM, BAC, WFC, C?

> Pull a governance snapshot for Equinix and tell me what a governance analyst would flag.

## Cross-company comparison

> Compare CEO compensation and pay-for-performance alignment across the three largest US healthcare REITs in coverage.

> I'm preparing for a compensation committee meeting. Compare our client's CEO pay structure against their disclosed peer group and highlight anything an investor would push back on.

## Peer-group scrutiny

> Who does Crowdstrike benchmark itself against, and is that peer group defensible?

> Price a peer-cohort audit for Vici Properties.

## What it will not do

- It will not answer for a company outside coverage — it returns `not_in_coverage` rather than guessing.
- It will not give investment advice.
- Narrative outputs are grounded against the underlying figures; if a claim can't be supported by the data, the server declines rather than improvising.
