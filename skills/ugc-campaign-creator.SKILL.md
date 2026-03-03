---
name: ugc-campaign-creator
description: Create UGC (User-Generated Content) video ad campaigns on Meta from a library of video assets, with proper ad set structure and audience targeting.
---

# UGC Campaign Creator

## Purpose

Build Meta ad campaigns featuring UGC video creatives. This skill handles the full workflow: uploading video assets, creating campaigns with multiple ad sets for A/B testing, and attaching video creatives with proper ad copy. UGC ads typically outperform polished brand content for direct-response objectives.

## When to Use

- The user has UGC video files and wants to run them as Meta ads.
- The user says "create UGC campaign", "launch video ads", or "set up UGC ads".
- Multiple video creatives need to be tested across different audiences.
- The user wants to create single-video ad sets within an ABO (Ad Set Budget Optimization) structure.

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

1. **Gather video assets.** Identify the video files:
   - Look for videos in a local directory (e.g., `media/ugc/`, `Media Assets/`)
   - Accept video URLs if already hosted
   - Supported formats: MP4, MOV (Meta recommends MP4, H.264, max 4GB)
   - Optimal specs: 9:16 for Stories/Reels, 1:1 for Feed, 4:5 for Feed vertical

2. **Upload videos to Meta** (if local files):
   ```
   POST /act_{ad_account_id}/advideos
   Content-Type: multipart/form-data
   source: @video.mp4
   title: "{video_title}"
   ```
   Save the returned `id` for each video. Video processing takes time; poll status:
   ```
   GET /{video_id}?fields=status
   ```
   Wait for `status.video_status === "ready"`.

3. **Map videos to products** (optional). If the UGC features specific products, load or create a `video-product-map.json`:
   ```json
   {
     "video_filename.mp4": {
       "video_id": "123456",
       "product_ids": ["prod_1", "prod_2"],
       "description": "Unboxing seasonal footwear"
     }
   }
   ```

4. **Create the campaign.** Use ABO (Ad Set Budget Optimization) for UGC testing:
   ```
   POST /act_{ad_account_id}/campaigns
   {
     "name": "UGC - {campaign_name}",
     "objective": "OUTCOME_SALES",
     "status": "PAUSED",
     "special_ad_categories": []
   }
   ```
   Note: Do NOT enable CBO for UGC testing — use individual ad set budgets to control spend per creative.

5. **Create one ad set per video** (or per audience segment):
   ```
   POST /act_{ad_account_id}/adsets
   {
     "name": "UGC - {video_name} - {audience}",
     "campaign_id": "{campaign_id}",
     "daily_budget": {budget_minor currency units (e.g., cents)},
     "billing_event": "IMPRESSIONS",
     "optimization_goal": "OFFSITE_CONVERSIONS",
     "promoted_object": {
       "pixel_id": "{pixel_id}",
       "custom_event_type": "PURCHASE"
     },
     "targeting": {
       "geo_locations": {"countries": ["{your_country_code}"]},
       "age_min": 18,
       "age_max": 65,
       "publisher_platforms": ["facebook", "instagram"],
       "facebook_positions": ["feed", "video_feeds", "story", "reels"],
       "instagram_positions": ["stream", "story", "reels", "explore"]
     },
     "status": "PAUSED"
   }
   ```

6. **Create the video ad creative.**
   ```
   POST /act_{ad_account_id}/adcreatives
   {
     "name": "UGC Creative - {video_name}",
     "object_story_spec": {
       "page_id": "{page_id}",
       "video_data": {
         "video_id": "{video_id}",
         "message": "{primary_text}",
         "title": "{headline}",
         "link_description": "{description}",
         "call_to_action": {
           "type": "SHOP_NOW",
           "value": {"link": "https://your-store.myshopify.com"}
         },
         "image_hash": "{thumbnail_hash}"
       }
     },
     "instagram_actor_id": "{instagram_id}"
   }
   ```

7. **Create the ad.**
   ```
   POST /act_{ad_account_id}/ads
   {
     "name": "UGC Ad - {video_name}",
     "adset_id": "{adset_id}",
     "creative": {"creative_id": "{creative_id}"},
     "status": "PAUSED"
   }
   ```

8. **Report the full structure.** Print a table showing campaign > ad sets > ads with all IDs.

## Required Environment Variables

- `META_ACCESS_TOKEN` — long-lived User Access Token
- `META_AD_ACCOUNT_ID` — ad account ID
- `META_PIXEL_ID` — pixel ID
- `META_PAGE_ID` — Facebook Page ID
- `META_INSTAGRAM_ID` — Instagram account ID

## Example Usage

```
User: Create a UGC campaign with these 5 videos, 200 {currency} per ad set, targeting men 25-45 in your market
```

The skill will upload videos, create the campaign with 5 ad sets (one per video), and attach the creatives.
