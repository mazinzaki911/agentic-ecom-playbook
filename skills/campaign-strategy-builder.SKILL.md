---
name: campaign-strategy-builder
description: Build complete campaign strategies including creative-to-link matching, testing plans, budget allocation, and analysis frameworks for Meta ads.
---

# Campaign Strategy Builder

## Purpose

Design and document complete Meta advertising campaign strategies before execution. This skill focuses on the strategic planning layer — mapping creatives to landing pages, designing A/B testing matrices, allocating budgets across campaign components, and building analysis frameworks to measure success. The output is a detailed strategy document that the media-buying-sop-pack skills can then execute.

## When to Use

- The user says "plan a campaign", "build a campaign strategy", "design my ad plan", or "map creatives to landing pages".
- The user runs `/campaign-strategy`.
- A new campaign initiative needs strategic planning before creation.
- The user wants to structure a testing plan for different creative angles.

## Prerequisites

- **Playwright MCP server** must be installed and running. This is used to research current Meta ad formats, placement options, and best practices.
- Meta Marketing API access token in `.env` (for pulling historical performance data)
- Ad Account ID configured

## Self-Learning Protocol

Before building a strategy, use Playwright to verify current platform capabilities:

1. **Navigate to the official docs:**
   - Meta Ads Guide: `https://www.facebook.com/business/ads-guide`
   - Creative Best Practices: `https://www.facebook.com/business/help/1602339903334508`
   - Ad Format Specs: `https://www.facebook.com/business/help/980593475366490`
   - Marketing API Reference: `https://developers.facebook.com/docs/marketing-api/reference/`

2. **Check for new ad formats or placements.** Meta frequently adds new creative formats and placement options.

3. **Review current targeting capabilities.** Targeting options change over time (e.g., interest targeting restrictions, Advantage+ audience updates).

## Instructions

### `/campaign-strategy` — Build Campaign Strategy

1. **Gather campaign context:**
   - Campaign objective (sales, leads, awareness, traffic)
   - Product or collection being promoted
   - Target audience description
   - Available creative assets (images, videos, UGC)
   - Landing pages / destination URLs
   - Total budget and duration
   - Any seasonal or promotional context

2. **Map creatives to landing pages:**
   ```
   Creative-to-Link Matrix
   =======================

   Creative 1: Hero lifestyle image
   ├── Landing: /collections/{collection-handle}
   ├── Audience: Prospecting (broad)
   └── CTA: Shop Now

   Creative 2: Product carousel (4 products)
   ├── Landing: /collections/{collection-handle}
   ├── Audience: Retargeting (website visitors)
   └── CTA: Shop Now

   Creative 3: UGC video testimonial
   ├── Landing: /products/{product-handle}
   ├── Audience: Prospecting (lookalike)
   └── CTA: Learn More

   Creative 4: DPA (dynamic product ads)
   ├── Landing: Dynamic (product page)
   ├── Audience: Retargeting (cart abandoners)
   └── CTA: Shop Now
   ```

3. **Design the testing plan:**
   ```
   A/B Testing Matrix
   ==================

   Test 1: Creative Format
   ├── Variable: Image vs. Video vs. Carousel
   ├── Control: Hero image (Creative 1)
   ├── Variants: Video (Creative 3), Carousel (Creative 2)
   ├── Audience: Same prospecting audience
   ├── Budget: Equal split
   ├── Duration: 7 days minimum
   └── Success Metric: ROAS

   Test 2: Audience
   ├── Variable: Interest vs. Lookalike vs. Broad
   ├── Creative: Winner from Test 1
   ├── Budget: Equal split
   ├── Duration: 7 days
   └── Success Metric: CPA
   ```

4. **Allocate budget:**
   ```
   Budget Allocation
   =================
   Total: {budget} {currency} over {duration} days

   Phase 1 — Testing (Days 1-7): 30% of budget
   ├── Test 1: {amount} {currency}/day split across 3 ad sets
   └── Test 2: {amount} {currency}/day split across 3 ad sets

   Phase 2 — Optimization (Days 8-14): 30% of budget
   ├── Scale winners from Phase 1
   └── Kill losers (ROAS < 1x)

   Phase 3 — Scaling (Days 15+): 40% of budget
   ├── Top 2 performing ad sets get 70% of Phase 3 budget
   └── Exploration (new angles): 30% of Phase 3 budget
   ```

5. **Define the analysis framework:**
   ```
   Analysis Framework
   ==================

   Daily Checks:
   - Spend pacing (on track vs. over/under)
   - ROAS by ad set
   - Frequency (pause if > 3 for prospecting)

   Weekly Analysis:
   - Winner/loser identification
   - Budget reallocation recommendations
   - Creative fatigue detection (CTR decline > 20%)

   Campaign Review (end of duration):
   - Overall ROAS and revenue
   - Best performing: creative, audience, placement
   - Learnings for next campaign
   - Recommended next steps
   ```

6. **Generate the strategy document** as a comprehensive markdown file:
   ```
   Campaign Strategy: {campaign_name}
   ====================================
   Date: {date}
   Objective: {objective}
   Budget: {budget} {currency}
   Duration: {duration} days

   [Full strategy with all sections above]
   ```

7. **Save the strategy** to `docs/strategies/{campaign-name}-strategy.md`.

8. **Offer execution.** Ask if the user wants to execute the strategy using the media-buying-sop-pack skills.

## Required Environment Variables

- `META_ACCESS_TOKEN` — long-lived User Access Token (for pulling historical data)
- `META_AD_ACCOUNT_ID` — ad account ID

## Example Usage

```
User: /campaign-strategy for a spring collection launch with 15,000 {currency} budget over 30 days
```

The skill will gather details about the collection, available creatives, and target audience, then produce a complete strategy document.

```
User: Build a campaign strategy for retargeting cart abandoners with UGC video content
```

The skill will design a focused retargeting strategy with UGC creative mapping and testing plan.
