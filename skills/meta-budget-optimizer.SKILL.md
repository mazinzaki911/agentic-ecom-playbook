---
name: meta-budget-optimizer
description: Analyze Meta campaign performance metrics (ROAS, CPA, CPM) and rebalance budgets across campaigns and ad sets for optimal spend allocation.
---

# Meta Budget Optimizer

## Purpose

Pull performance data from Meta Ads Insights API, analyze ROAS/CPA/CPM across campaigns and ad sets, and recommend or apply budget reallocations. This skill helps media buyers make data-driven budget decisions instead of manual spreadsheet analysis.

## When to Use

- The user asks "how are my campaigns performing?" or "which ad sets should I scale?"
- The user wants to rebalance budgets based on ROAS or CPA.
- The user says "optimize budgets", "analyze campaign performance", or "reallocate spend".
- A weekly/monthly budget review is needed.
- The user wants to identify underperforming ad sets to pause.

## Instructions

1. **Fetch campaign performance data.** Query the Insights API:
   ```
   GET /act_{ad_account_id}/insights?fields=campaign_name,campaign_id,spend,purchase_roas,actions,cost_per_action_type,impressions,cpm,ctr,frequency&time_range={"since":"2026-02-24","until":"2026-03-03"}&level=campaign&filtering=[{"field":"campaign.delivery_info","operator":"IN","value":["active","recently_completed"]}]
   ```

2. **Fetch ad set level data** for deeper analysis:
   ```
   GET /act_{ad_account_id}/insights?fields=adset_name,adset_id,campaign_name,spend,purchase_roas,actions,cost_per_action_type,impressions,cpm,ctr,frequency&time_range={"since":"2026-02-24","until":"2026-03-03"}&level=adset&filtering=[{"field":"adset.delivery_info","operator":"IN","value":["active"]}]
   ```

3. **Parse the metrics.** Extract key performance indicators:
   ```js
   const metrics = insights.map(row => ({
     name: row.campaign_name || row.adset_name,
     id: row.campaign_id || row.adset_id,
     spend: parseFloat(row.spend || 0),
     roas: parseFloat(row.purchase_roas?.[0]?.value || 0),
     purchases: row.actions?.find(a => a.action_type === 'purchase')?.value || 0,
     cpa: parseFloat(row.cost_per_action_type?.find(a => a.action_type === 'purchase')?.value || 0),
     cpm: parseFloat(row.cpm || 0),
     ctr: parseFloat(row.ctr || 0),
     frequency: parseFloat(row.frequency || 0)
   }));
   ```

4. **Analyze and categorize.** Classify each campaign/ad set:
   - **Scale** (ROAS > 3x, CPA within target): Increase budget 20-30%
   - **Maintain** (ROAS 2-3x): Keep current budget
   - **Reduce** (ROAS 1-2x, high frequency): Decrease budget 20-30%
   - **Pause** (ROAS < 1x for 7+ days, or frequency > 6): Recommend pausing

5. **Calculate recommended budgets.** For CBO campaigns, adjust at campaign level. For ABO, adjust per ad set:
   ```js
   function recommendBudget(current, roas, targetRoas = 3) {
     if (roas >= targetRoas * 1.5) return Math.round(current * 1.3);  // Scale aggressively
     if (roas >= targetRoas) return Math.round(current * 1.2);        // Scale moderately
     if (roas >= targetRoas * 0.66) return current;                   // Maintain
     if (roas >= 1) return Math.round(current * 0.7);                 // Reduce
     return 0; // Recommend pause
   }
   ```

6. **Present the analysis.** Format as a clear report:
   ```
   Campaign Performance Report (Feb 24 - Mar 3, 2026)
   ====================================================
   Campaign                  | Spend   | ROAS | CPA    | Action
   DPA Retargeting          | 2,500   | 4.2x | 180    | SCALE (+30%)
   Winter Collection CBO    | 1,800   | 3.1x | 220    | SCALE (+20%)
   UGC Testing              | 800     | 1.5x | 450    | REDUCE (-30%)
   Awareness - Location     | 500     | 0.3x | 2,100  | PAUSE

   Recommended total daily budget: 5,600 EGP (current: 5,600 EGP)
   ```

7. **Apply budget changes** (only if the user confirms):
   ```
   POST /{campaign_id}
   { "daily_budget": {new_budget_piasters} }
   ```
   Or for ad sets:
   ```
   POST /{adset_id}
   { "daily_budget": {new_budget_piasters} }
   ```
   Note: Budgets in the API are in piasters (multiply EGP by 100).

8. **Pause underperformers** (only if the user confirms):
   ```
   POST /{adset_id}
   { "status": "PAUSED" }
   ```

## Required Environment Variables

- `META_ACCESS_TOKEN` — long-lived User Access Token
- `META_AD_ACCOUNT_ID` — ad account ID (e.g., `act_7781591668619406`)

## Key Metrics Reference

| Metric | Good (Fashion/EG) | Warning | Critical |
|--------|-------------------|---------|----------|
| ROAS | > 3x | 1.5-3x | < 1.5x |
| CPA | < 250 EGP | 250-500 EGP | > 500 EGP |
| CPM | < 80 EGP | 80-150 EGP | > 150 EGP |
| CTR | > 1.5% | 1-1.5% | < 1% |
| Frequency | < 3 (cold) | 3-5 | > 5 |

## Example Usage

```
User: Analyze last 7 days of campaign performance and recommend budget changes
```

The skill will pull insights, analyze ROAS/CPA, categorize campaigns, and present a budget reallocation plan.
