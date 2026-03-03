---
name: media-buying-sop-pack
description: Collection of 5 Meta ads SOPs — campaign creation, ad set configuration, ad creative, performance monitoring, and campaign scaling.
---

# Media Buying SOP Pack

## Purpose

Provide a complete set of Standard Operating Procedures for managing Meta (Facebook/Instagram) advertising campaigns. This pack covers the full campaign lifecycle from creation through scaling, with each SOP executable as a standalone command or chained together for end-to-end campaign management.

## When to Use

- The user wants to create or manage Meta ad campaigns.
- The user runs `/create-campaign`, `/setup-adset`, `/create-ad`, `/check-performance`, or `/scale-campaign`.
- A new campaign needs to be launched end-to-end.
- Performance needs to be reviewed and optimized.

## Prerequisites

- **Playwright MCP server** must be installed and running. This is what makes these SOPs future-proof — they can navigate to live Meta documentation to verify current API endpoints, parameters, and best practices.
- Meta Marketing API access token in `.env`
- Ad Account ID, Pixel ID, Page ID configured

## Self-Learning Protocol

Before executing any API call that fails or seems outdated, use Playwright to verify:

1. **Navigate to the official docs:**
   - Meta Marketing API: `https://developers.facebook.com/docs/marketing-api/reference/`
   - Campaign Reference: `https://developers.facebook.com/docs/marketing-api/reference/ad-campaign-group/`
   - Ad Set Reference: `https://developers.facebook.com/docs/marketing-api/reference/ad-campaign/`
   - Ad Creative Reference: `https://developers.facebook.com/docs/marketing-api/reference/ad-creative/`
   - Insights Reference: `https://developers.facebook.com/docs/marketing-api/reference/ad-account/insights/`
   - Graph API Changelog: `https://developers.facebook.com/docs/graph-api/changelog`

2. **Check the current API version.** Verify the configured version is still supported.

3. **Read the endpoint reference.** Verify parameter names, enum values, and required fields.

4. **Adapt and retry.** Update the API call with current parameters. Save learnings to CLAUDE.md.

## Instructions

### SOP 1: `/create-campaign` — Create a Campaign

1. **Gather inputs:**
   - Campaign name
   - Objective: `OUTCOME_SALES`, `OUTCOME_LEADS`, `OUTCOME_AWARENESS`, `OUTCOME_TRAFFIC`, `OUTCOME_ENGAGEMENT`
   - Budget type: CBO (Campaign Budget Optimization) or ABO (Ad Set Budget)
   - Budget amount (in local currency)
   - Special ad categories (if any)

2. **Create the campaign:**
   ```
   POST /act_{ad_account_id}/campaigns
   {
     "name": "{campaign_name}",
     "objective": "{objective}",
     "status": "PAUSED",
     "special_ad_categories": [],
     "daily_budget": {budget_in_minor_units}  // only for CBO
   }
   ```

3. **Report the campaign ID** and link to Ads Manager.

### SOP 2: `/setup-adset` — Configure an Ad Set

1. **Gather inputs:**
   - Campaign ID (from SOP 1)
   - Ad set name
   - Budget (if ABO)
   - Targeting: location, age, gender, interests, custom audiences
   - Placement: automatic or manual
   - Optimization goal: `OFFSITE_CONVERSIONS`, `LINK_CLICKS`, `IMPRESSIONS`, `REACH`
   - Schedule: start date, end date (optional)

2. **Create the ad set:**
   ```
   POST /act_{ad_account_id}/adsets
   {
     "name": "{adset_name}",
     "campaign_id": "{campaign_id}",
     "daily_budget": {budget_in_minor_units},
     "billing_event": "IMPRESSIONS",
     "optimization_goal": "{optimization_goal}",
     "promoted_object": {
       "pixel_id": "{pixel_id}",
       "custom_event_type": "PURCHASE"
     },
     "targeting": {
       "geo_locations": {"countries": ["{country_code}"]},
       "age_min": {min_age},
       "age_max": {max_age}
     },
     "status": "PAUSED"
   }
   ```

3. **Report the ad set ID.**

### SOP 3: `/create-ad` — Create Ad Creative and Ad

1. **Gather inputs:**
   - Ad set ID (from SOP 2)
   - Creative type: single image, carousel, video, collection
   - Media (image URLs, video URLs, or hashes)
   - Primary text, headline, description
   - CTA type: `SHOP_NOW`, `LEARN_MORE`, `SIGN_UP`, etc.
   - Destination URL

2. **Upload media** (if local files):
   ```
   POST /act_{ad_account_id}/adimages
   ```

3. **Create the creative:**
   ```
   POST /act_{ad_account_id}/adcreatives
   {
     "name": "{creative_name}",
     "object_story_spec": {
       "page_id": "{page_id}",
       "link_data": {
         "message": "{primary_text}",
         "link": "{destination_url}",
         "name": "{headline}",
         "call_to_action": {"type": "{cta_type}"},
         "image_hash": "{image_hash}"
       }
     }
   }
   ```

4. **Create the ad:**
   ```
   POST /act_{ad_account_id}/ads
   {
     "name": "{ad_name}",
     "adset_id": "{adset_id}",
     "creative": {"creative_id": "{creative_id}"},
     "status": "PAUSED"
   }
   ```

### SOP 4: `/check-performance` — Monitor Performance

1. **Fetch insights** for the specified time range:
   ```
   GET /act_{ad_account_id}/insights?fields=spend,impressions,clicks,ctr,cpc,cpm,actions,cost_per_action_type,purchase_roas&time_range={"since":"{start}","until":"{end}"}&level=campaign
   ```

2. **Parse and format** the key metrics:
   - Spend, Impressions, Clicks, CTR, CPC, CPM
   - Purchases, Cost per Purchase, ROAS
   - Add to Cart, Initiate Checkout (funnel metrics)

3. **Flag anomalies:**
   - ROAS < 1x (losing money)
   - Frequency > 3 (audience fatigue for prospecting)
   - CTR < 0.5% (creative fatigue)
   - CPM spike > 20% vs. previous period

4. **Recommend actions** based on findings.

### SOP 5: `/scale-campaign` — Scale Campaign

1. **Analyze current performance** (invoke SOP 4).

2. **Determine scaling strategy:**
   - **Vertical scaling**: Increase budget by 20% (safe) or 50% (aggressive).
   - **Horizontal scaling**: Duplicate winning ad sets with new audiences.
   - **Creative scaling**: Add new creatives to winning ad sets.

3. **Execute the chosen strategy:**
   - For budget increase: `POST /{adset_id}` with new `daily_budget`.
   - For duplication: Create new ad set with same creative but different targeting.
   - For creative addition: Add new ads to existing ad set.

4. **Set monitoring schedule.** Check performance 24h and 72h after scaling.

## Required Environment Variables

- `META_ACCESS_TOKEN` — long-lived User Access Token
- `META_AD_ACCOUNT_ID` — ad account ID
- `META_PIXEL_ID` — pixel ID
- `META_PAGE_ID` — Facebook Page ID
- `META_INSTAGRAM_ID` — Instagram account ID

## Example Usage

```
User: /create-campaign sales campaign with 500 {currency}/day budget
```

```
User: /check-performance last 7 days
```

```
User: /scale-campaign increase budget on winning ad sets by 20%
```
