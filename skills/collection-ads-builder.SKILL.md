---
name: collection-ads-builder
description: Build Meta collection ad campaigns with a hero image/video, product catalog grid, and Instant Experience (Canvas) destination.
---

# Collection Ads Builder

## Purpose

Create collection ad campaigns on Meta that combine a hero image or video with a product grid from the catalog. Collection ads drive users to an Instant Experience (full-screen mobile storefront) before sending them to the website. This format is highly effective for showcasing product categories.

## When to Use

- The user wants to create collection ads for a product category or seasonal campaign.
- The user says "build collection ads", "create carousel collection campaign", or "set up Instant Experience ads".
- A new collection (e.g., "Spring Loafers", "Winter Boots") needs a dedicated ad campaign.

## Instructions

1. **Gather inputs.** Confirm:
   - Collection/campaign name
   - Hero image or video (URL or local file path)
   - Product set ID or criteria for the product grid (4+ products shown beneath the hero)
   - Primary text, headline, and CTA
   - Budget (daily or lifetime, in EGP)
   - Target audience: broad, interest-based, or retargeting

2. **Create the Instant Experience (Canvas).**
   ```
   POST /act_{ad_account_id}/adcanvases
   {
     "name": "IX - {collection_name}",
     "body_elements": [
       {
         "element_type": "HEADER",
         "image_hash": "{hero_image_hash}"
       },
       {
         "element_type": "PRODUCT_SET",
         "product_set_id": "{product_set_id}",
         "max_products": 50
       },
       {
         "element_type": "BUTTON",
         "button_text": "Shop Now",
         "button_url": "https://achilles-stores.com/collections/{handle}"
       }
     ]
   }
   ```

3. **Upload the hero image** (if not already uploaded):
   ```
   POST /act_{ad_account_id}/adimages
   Content-Type: multipart/form-data
   filename: @hero-image.jpg
   ```
   Save the returned `hash` for the creative.

4. **Create the campaign.**
   ```
   POST /act_{ad_account_id}/campaigns
   {
     "name": "Collection - {collection_name}",
     "objective": "OUTCOME_SALES",
     "status": "PAUSED",
     "special_ad_categories": []
   }
   ```

5. **Create the ad set** with targeting:
   ```
   POST /act_{ad_account_id}/adsets
   {
     "name": "Collection AdSet - {collection_name}",
     "campaign_id": "{campaign_id}",
     "daily_budget": {budget_piasters},
     "billing_event": "IMPRESSIONS",
     "optimization_goal": "OFFSITE_CONVERSIONS",
     "promoted_object": {
       "product_set_id": "{product_set_id}",
       "custom_event_type": "PURCHASE",
       "pixel_id": "{pixel_id}"
     },
     "targeting": {
       "geo_locations": {"countries": ["EG"]},
       "age_min": 18,
       "age_max": 65,
       "publisher_platforms": ["facebook", "instagram"]
     },
     "status": "PAUSED"
   }
   ```

6. **Create the collection ad creative.**
   ```
   POST /act_{ad_account_id}/adcreatives
   {
     "name": "Collection Creative - {collection_name}",
     "object_story_spec": {
       "page_id": "{page_id}",
       "link_data": {
         "message": "{primary_text}",
         "link": "https://achilles-stores.com/collections/{handle}",
         "name": "{headline}",
         "call_to_action": {"type": "SHOP_NOW"},
         "retailer_item_ids": ["{variant_id_1}", "{variant_id_2}", "{variant_id_3}", "{variant_id_4}"],
         "image_hash": "{hero_image_hash}"
       }
     },
     "product_set_id": "{product_set_id}",
     "instagram_actor_id": "{instagram_account_id}"
   }
   ```

7. **Create the ad.**
   ```
   POST /act_{ad_account_id}/ads
   {
     "name": "Collection Ad - {collection_name}",
     "adset_id": "{adset_id}",
     "creative": {"creative_id": "{creative_id}"},
     "status": "PAUSED"
   }
   ```

8. **Report results.** Provide all created object IDs and a link to review in Ads Manager.

## Required Environment Variables

- `META_ACCESS_TOKEN` — long-lived User Access Token
- `META_AD_ACCOUNT_ID` — ad account ID
- `META_CATALOG_ID` — product catalog ID
- `META_PIXEL_ID` — pixel ID
- `META_PAGE_ID` — Facebook Page ID
- `META_INSTAGRAM_ID` — Instagram account ID

## Example Usage

```
User: Create a collection ad campaign for the Women's Winter Boots collection with this hero image and 300 EGP daily budget
```

The skill will upload the hero image, create the Instant Experience, campaign, ad set, creative, and ad.
