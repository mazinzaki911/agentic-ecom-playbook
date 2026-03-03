---
name: dpa-campaign-launcher
description: Launch Dynamic Product Ad (DPA) campaigns on Meta, including catalog sales campaigns with retargeting and prospecting ad sets.
---

# DPA Campaign Launcher

## Purpose

Create and launch Dynamic Product Ad campaigns on Meta Ads Manager via the Marketing API. DPA campaigns use the product catalog to automatically show relevant products to users based on their browsing behavior (retargeting) or predicted interest (prospecting/broad audience).

## When to Use

- The user wants to create a new DPA or catalog sales campaign.
- The user says "launch DPA", "create retargeting campaign", or "set up catalog ads".
- A new product collection needs its own DPA campaign.
- The user wants to add prospecting (broad audience) ad sets to an existing DPA campaign.

## Prerequisites

- **Playwright MCP server** must be installed and running. This is what makes this skill future-proof — it can navigate to live documentation to verify current API endpoints, parameters, and behavior before executing.
- Meta Marketing API access token in `.env`
- Pixel ID and Ad Account ID configured

## Self-Learning Protocol

Before executing any API call that fails or seems outdated, use Playwright to verify:

1. **Navigate to the official docs:**
   - Meta Marketing API: `https://developers.facebook.com/docs/marketing-api/reference/`
   - Meta Catalog API: `https://developers.facebook.com/docs/marketing-api/reference/product-catalog/`
   - Graph API changelog: `https://developers.facebook.com/docs/graph-api/changelog`

2. **Check the current API version.** Meta deprecates versions regularly. If a call returns "unknown endpoint" or "deprecated", visit the changelog to find the replacement.

3. **Read the endpoint reference.** Navigate to the specific endpoint page, extract the current parameters, required fields, and any new restrictions.

4. **Adapt and retry.** Update the API call with current parameters and re-execute. Save what you learned to CLAUDE.md memory so future runs start with the correct info.

This means the skill never goes stale — even if Meta ships a breaking change tomorrow, Claude Code can read the docs and figure out the new way.

## Instructions

1. **Gather campaign parameters.** Confirm with the user:
   - Campaign name and objective (default: `OUTCOME_SALES`)
   - Daily or lifetime budget (in EGP, specified in piasters: multiply by 100)
   - Product set or catalog segment to advertise
   - Audience type: retargeting (website visitors, cart abandoners) or prospecting (broad/lookalike)
   - Optimization goal (default: `OFFSITE_CONVERSIONS`, event: `Purchase`)

2. **Create the campaign.**
   ```
   POST /act_{ad_account_id}/campaigns
   {
     "name": "{campaign_name}",
     "objective": "OUTCOME_SALES",
     "status": "PAUSED",
     "special_ad_categories": [],
     "campaign_budget_optimization": true,
     "daily_budget": {budget_in_piasters}
   }
   ```
   Always create campaigns in PAUSED status first so the user can review before going live.

3. **Create the product set** (if needed). If the user wants to target a specific collection:
   ```
   POST /{catalog_id}/product_sets
   {
     "name": "{product_set_name}",
     "filter": {
       "product_type": {"i_contains": "{collection_keyword}"}
     }
   }
   ```
   Alternatively, filter by `retailer_id` for explicit product lists.

4. **Create the retargeting ad set.**
   ```
   POST /act_{ad_account_id}/adsets
   {
     "name": "DPA Retargeting - {segment}",
     "campaign_id": "{campaign_id}",
     "billing_event": "IMPRESSIONS",
     "optimization_goal": "OFFSITE_CONVERSIONS",
     "promoted_object": {
       "product_set_id": "{product_set_id}",
       "custom_event_type": "PURCHASE",
       "pixel_id": "{pixel_id}"
     },
     "targeting": {
       "product_audience_specs": [{
         "product_set_id": "{product_set_id}",
         "inclusions": [
           {"retention_seconds": 1209600, "rule": {"event": {"eq": "ViewContent"}}},
           {"retention_seconds": 1209600, "rule": {"event": {"eq": "AddToCart"}}}
         ],
         "exclusions": [
           {"retention_seconds": 604800, "rule": {"event": {"eq": "Purchase"}}}
         ]
       }],
       "geo_locations": {"countries": ["EG"]}
     },
     "status": "PAUSED"
   }
   ```
   Key retargeting windows: ViewContent 14 days, AddToCart 14 days, exclude Purchase 7 days.

5. **Create the prospecting ad set** (optional). For broad audience DPA:
   ```
   POST /act_{ad_account_id}/adsets
   {
     "name": "DPA Prospecting - {segment}",
     "campaign_id": "{campaign_id}",
     "targeting": {
       "product_audience_specs": [{
         "product_set_id": "{product_set_id}",
         "inclusions": [],
         "exclusions": [
           {"retention_seconds": 2592000, "rule": {"event": {"eq": "Purchase"}}}
         ]
       }],
       "geo_locations": {"countries": ["EG"]},
       "age_min": 18,
       "age_max": 65
     },
     "status": "PAUSED"
   }
   ```

6. **Create the ad creative.** DPA uses template creatives:
   ```
   POST /act_{ad_account_id}/adcreatives
   {
     "name": "DPA Creative - {segment}",
     "product_set_id": "{product_set_id}",
     "object_story_spec": {
       "page_id": "{page_id}",
       "template_data": {
         "message": "Shop {{product.name}} now!",
         "link": "https://achilles-stores.com",
         "call_to_action": {"type": "SHOP_NOW"}
       }
     }
   }
   ```

7. **Create the ad** linking creative to ad set:
   ```
   POST /act_{ad_account_id}/ads
   {
     "name": "DPA Ad - {segment}",
     "adset_id": "{adset_id}",
     "creative": {"creative_id": "{creative_id}"},
     "status": "PAUSED"
   }
   ```

8. **Report the campaign structure.** Print a summary with IDs for campaign, ad sets, creatives, and ads. Remind the user to review in Ads Manager before activating.

## Required Environment Variables

- `META_ACCESS_TOKEN` — long-lived User Access Token
- `META_AD_ACCOUNT_ID` — ad account ID (e.g., `act_7781591668619406`)
- `META_CATALOG_ID` — product catalog ID
- `META_PIXEL_ID` — pixel ID for conversion tracking
- `META_PAGE_ID` — Facebook Page ID

## Example Usage

```
User: Launch a DPA retargeting campaign for the winter boots collection with 500 EGP daily budget
```

The skill will create the campaign, product set, retargeting ad set, creative, and ad — all in PAUSED status for review.
