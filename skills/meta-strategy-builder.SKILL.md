---
name: meta-strategy-builder
description: Build complete Meta advertising strategies including funnel structure, audience segmentation, creative testing plans, and budget allocation.
---

# Meta Strategy Builder

## Purpose

Build comprehensive Meta advertising strategies from the ground up. This skill goes beyond individual campaign creation to design the full advertising architecture — funnel stages, audience segmentation, creative testing frameworks, budget allocation across the funnel, and KPI targets. It produces a strategic document that can then be executed using the media-buying-sop-pack skills.

## When to Use

- The user says "build a Meta strategy", "plan my ad funnel", "design my advertising approach", or "create an ad strategy".
- The user runs `/meta-strategy test`, `/meta-strategy scale`, `/meta-strategy funnel`, or `/meta-strategy audit`.
- A new season, product launch, or business phase requires a fresh advertising approach.
- The user wants to restructure their existing Meta ad account.

## Prerequisites

- **Playwright MCP server** must be installed and running. This is used to research current Meta best practices, feature announcements, and competitive benchmarks.
- Meta Marketing API access token in `.env`
- Ad Account ID configured
- Ideally, historical performance data (the skill can pull this via the Insights API)

## Self-Learning Protocol

Before building a strategy, use Playwright to research current best practices:

1. **Navigate to Meta's business resources:**
   - Meta Business Help Center: `https://www.facebook.com/business/help`
   - Meta Marketing API Best Practices: `https://developers.facebook.com/docs/marketing-api/best-practices/`
   - Meta Ads Guide: `https://www.facebook.com/business/ads-guide`

2. **Check for new features.** Meta frequently launches new campaign types, targeting options, and optimization features. Read the changelog to find anything relevant.

3. **Review industry benchmarks.** Navigate to industry benchmark reports to set realistic KPI targets for the user's vertical.

## Instructions

### `/meta-strategy funnel` — Build Full Funnel Strategy

1. **Gather business context:**
   - Product category / vertical
   - Average order value (AOV)
   - Monthly ad budget
   - Current customer base size
   - Geographic market(s)
   - Seasonal considerations

2. **Design the funnel structure:**
   ```
   TOFU (Top of Funnel) — Awareness/Prospecting
   ├── Broad targeting (interest-based)
   ├── Lookalike audiences (1-3%)
   └── Budget: 60% of total

   MOFU (Middle of Funnel) — Consideration
   ├── Website visitors (7-30 days)
   ├── Video viewers (25-75%)
   ├── Engaged social followers
   └── Budget: 20% of total

   BOFU (Bottom of Funnel) — Conversion
   ├── Add-to-cart abandoners (7 days)
   ├── Past purchasers (cross-sell)
   ├── High-intent website visitors
   └── Budget: 20% of total
   ```

3. **Define audience segments** for each funnel stage with exclusions to prevent overlap.

4. **Plan creative strategy:**
   - TOFU: Brand story, lifestyle, social proof
   - MOFU: Product features, reviews, comparisons
   - BOFU: Urgency, offers, DPA (Dynamic Product Ads)

5. **Set KPI targets** by funnel stage:
   | Stage | Primary KPI | Target |
   |-------|-------------|--------|
   | TOFU | CPM, Reach | CPM < market avg |
   | MOFU | CTR, CPC | CTR > 1.5% |
   | BOFU | ROAS, CPA | ROAS > 3x |

6. **Output the strategy document** with campaign names, budgets, audiences, and creative briefs.

### `/meta-strategy test` — Creative Testing Plan

1. **Design A/B test framework:**
   - Variable to test (creative, copy, audience, placement)
   - Control vs. variant setup
   - Sample size and duration requirements
   - Statistical significance threshold

2. **Output a testing calendar** with specific tests to run each week.

### `/meta-strategy scale` — Scaling Playbook

1. **Analyze current winners** (pull insights data).
2. **Design scaling plan** with vertical and horizontal scaling steps.
3. **Set guardrails** (max frequency, min ROAS, budget increase limits).

### `/meta-strategy audit` — Account Audit

1. **Pull all active campaigns, ad sets, and ads.**
2. **Analyze:** audience overlap, frequency distribution, budget allocation, creative fatigue.
3. **Generate recommendations** with priority ranking.

## Required Environment Variables

- `META_ACCESS_TOKEN` — long-lived User Access Token
- `META_AD_ACCOUNT_ID` — ad account ID
- `META_PIXEL_ID` — pixel ID

## Example Usage

```
User: /meta-strategy funnel for a fashion e-commerce brand with 10,000 {currency}/month budget
```

```
User: /meta-strategy audit — tell me what's wrong with my current ad account
```

```
User: /meta-strategy test — plan creative testing for next month
```
