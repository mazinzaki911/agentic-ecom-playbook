---
name: job-runner
description: Execute scheduled Shopify jobs (daily ops, weekly reports, custom tasks) by chaining multiple SOPs together.
---

# Job Runner

## Purpose

Execute pre-defined job sequences that chain multiple SOPs and skills together for routine e-commerce operations. Instead of running each step manually, the job runner orchestrates multi-step workflows for daily operations, weekly reporting, and custom batch tasks.

## When to Use

- The user says "run daily ops", "run weekly report", "run the morning job", or "execute batch task".
- The user runs `/shopify-job daily`, `/shopify-job weekly`, or `/shopify-job custom`.
- Routine operations need to be performed in a consistent sequence.

## Prerequisites

- **Playwright MCP server** must be installed and running. This is used by the underlying skills for self-learning when API calls need verification.
- Shopify Admin API access token in `.env`
- Meta Marketing API access token in `.env` (for jobs that touch Meta)
- All dependent skills must be available

## Self-Learning Protocol

The job runner delegates API calls to individual skills, each of which has its own self-learning protocol. However, the runner itself uses Playwright when:

1. **Verifying job dependencies.** Before chaining skills, check that the APIs they depend on are available.
   - Shopify API: `https://shopify.dev/docs/api/admin-graphql`
   - Meta API: `https://developers.facebook.com/docs/marketing-api/reference/`

2. **Checking for platform outages.** Before running a job sequence, verify that both platforms are operational.

## Instructions

### `/shopify-job daily` — Daily Operations

Run the daily operations sequence:

1. **Health Check** — Verify all connections are live.
   - Invokes: `health-check` skill
   - If any connection fails, stop and report.

2. **Check Orders** — Review new orders since last run.
   - Invokes: `/check-orders` from `shopify-jobs-sop-pack`
   - Filters: `created_at:>{last_run_date}`
   - Reports: new order count, total revenue, any issues.

3. **Check Inventory** — Scan for low/out-of-stock items.
   - Invokes: `/check-inventory` from `shopify-jobs-sop-pack`
   - Flags items below threshold.
   - If critical items are out of stock, alert.

4. **Check Performance** — Pull today's ad performance.
   - Invokes: `/check-performance` from `media-buying-sop-pack`
   - Reports: spend, ROAS, purchases, anomalies.

5. **Generate Daily Summary:**
   ```
   Daily Operations Report — {date}
   ==================================

   Orders
   ------
   New orders: 23
   Revenue: {amount} {currency}
   Unfulfilled: 8

   Inventory Alerts
   ----------------
   Low stock: 5 SKUs
   Out of stock: 2 SKUs

   Ad Performance
   --------------
   Spend: {amount} {currency}
   Purchases: 15
   ROAS: 3.2x

   Action Items
   ------------
   - [!] Restock SKU-1234 (0 units remaining)
   - [!] Creative fatigue detected on Ad Set "Summer Collection"
   ```

6. **Save the report** to `data/reports/daily-{date}.md`.

### `/shopify-job weekly` — Weekly Report

Run the weekly reporting sequence:

1. **Pull 7-day performance metrics** from both Shopify and Meta.
2. **Compare to previous week** — calculate week-over-week changes.
3. **Inventory trend analysis** — which SKUs are selling fastest, which are stagnant.
4. **Ad performance breakdown** — best/worst campaigns, ad sets, and creatives.
5. **Generate weekly summary** with trends and recommendations.
6. **Save the report** to `data/reports/weekly-{date}.md`.

### `/shopify-job custom` — Custom Job

1. **Accept a job definition** from the user:
   - List of skills/SOPs to chain
   - Parameters for each step
   - Conditions (stop on error, continue, skip)

2. **Execute the sequence** step by step, passing outputs between steps.

3. **Report results** for each step and overall.

## Required Environment Variables

- `SHOPIFY_STORE_URL` — Shopify store URL
- `SHOPIFY_ACCESS_TOKEN` — Admin API access token
- `META_ACCESS_TOKEN` — Meta access token
- `META_AD_ACCOUNT_ID` — ad account ID

## Example Usage

```
User: /shopify-job daily
```

Runs the full daily operations sequence and produces a summary report.

```
User: /shopify-job weekly
```

Generates a comprehensive weekly performance report.

```
User: /shopify-job custom — check inventory, update prices for low stock, then regenerate the Meta feed
```

Chains three skills together in a custom sequence.
