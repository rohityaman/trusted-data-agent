# trusted-data-agent
A portfolio project that builds a trust layer for Snowflake's AI agents, so business users can measure how accurate their answers are.

## The problem
While AI is everywhere today—and remarkably fast at analyzing data—taking its answers at face value can easily lead us down the wrong path. In Snowflake, for instance, business users can now ask questions about their data in plain English. However, they rarely have a way to verify if the answers are actually correct. Whether due to a weak semantic layer or a flawed analytics setup, the generated figures often look plausible enough that subtle errors slip right past notice.

This project bridges that trust gap. I measure accuracy against a set of verified answers, improve it step by step, and fail the build if it drops.

Consider a simple question like, "What was last quarter’s revenue?" In a dataset like TPC-H, there isn't just one answer—revenue could mean price before discount, after discount, or after discount plus tax, with or without returned items factored in. An AI agent that silently picks one definition isn't just making a guess; it is being confidently wrong to anyone expecting a different logic.

## Approach
The project is built in layers, and each layer is measured before the next one is added.

1. **Mart (M1):** Model TPC-H into 6–8 clean tables in dbt, with documented grain, key tests and three deliberately ambiguous metrics (revenue, returns, late shipments).
2. **Semantic views (M2):** Describe the mart's tables and metrics in Snowflake semantic views, so Cortex Analyst has the business definitions to answer from.
3. **Baseline (M2):** Write 40 questions with verified answers, run them through Snowflake's AI agent evaluation, and record the baseline accuracy. This number is the starting point for everything after.
4. **Raise accuracy (M3):** Improve descriptions, synonyms, verified queries and instructions, and log which change moved the number.
5. **Agent, governance and cost (M4–M6):** Add a Cortex Agent that routes between the semantic views and Cortex Search, check that roles can't see what they shouldn't, and measure cost per question.
6. **Regression guard (M7):** Run the eval in CI on every semantic view change, and fail the build if accuracy drops.

Every milestone ends in a number, and each number goes into the wins log.


## Dataset
Selected dataset is **TPC-H SF1** from `SNOWFLAKE_SAMPLE_DATA`. It is readily available in every Snowflake account, fast and cost-effective to query, yet complex enough to mirror real-world business ambiguity:

* **Revenue Definitions:** Raw price vs. post-discount vs. post-discount + tax.
* **Return Logic:** Including vs. excluding returned line items (`l_returnflag = 'R'`).
* **Fulfillment Metrics:** Varying logic for late shipments (e.g., `l_receiptdate > l_commitdate`).

In **M1**, I intentionally turn three of these edge cases into ambiguous metrics with documented, competing definitions to test how the AI handles business logic nuances.

## Milestones table
| # | Milestone | Done by | Done when |
|---|---|---|---|
| M0 | Setup | Fri 2026-10-23 | Account, resource monitor, GitHub repo, README with the problem statement |
| M1 | Mart | Fri 2026-12-18 | 6–8 tables over TPC-H **built in dbt (dbt Projects on Snowflake)**, grain documented, tests on keys, 3 deliberately ambiguous metrics defined |
| M2 | Semantic view + baseline | Fri 2027-03-12 | Domain-sliced semantic views (e.g. orders / lineitem revenue / customers) + Cortex Analyst; 40-question eval set with verified answers run through **Snowflake's AI agent evaluation**; baseline accuracy % |
| M3 | Raise accuracy | Fri 2027-04-23 | Descriptions, synonyms, verified queries, instructions; log which change moved the number. **Write-up #1** |
| M4 | Agent + MCP | Fri 2027-06-04 | Routing Cortex Agent over the semantic views + Cortex Search (small doc set: metric definitions), exposed through a Snowflake `CREATE MCP SERVER` and queried from Claude; routing and ambiguous-question cases added to evals |
| M4b | Open formats | Fri 2027-06-11 | ~3 sessions: one mart table as a Snowflake-managed Iceberg table, read from DuckDB or Spark; ADR "Iceberg vs native table: when" |
| M5 | Governance | Fri 2027-07-16 | 2 roles, row access + masking policies; eval cases proving the agent can't see what the role can't. **Write-up #2** |
| M6 | Observability & cost | Fri 2027-08-20 | Agent traces queried, cost per question measured. **Write-up #3** |
| M7 | Evals as CI | Fri 2027-10-15 | GitHub Action runs the eval on every semantic view change and fails on regression. **Write-up #4** |


## Stack + cost guardrail
Snowflake trial (Enterprise, AWS us-west-2), dbt, Cortex Analyst/Agents/Search, GitHub Actions. Resource monitor with a hard cap of about $50/month.

## Status
M0 done · M1 (dbt mart) next, due Fri 2026-12-18