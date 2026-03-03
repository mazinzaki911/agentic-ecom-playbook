---
name: meta-feed-sync
description: Sync product catalog feeds to Meta Commerce Manager via the Marketing API, handling supplementary feeds, variant IDs, and image updates.
---

# Meta Feed Sync

## Purpose

Upload and synchronize product catalog data to Meta Commerce Manager. This skill manages the complete feed pipeline: generating CSV/TSV feed files, filtering to synced-only products, uploading via the Catalog Batch API or Data Source API, and verifying ingestion status.

## When to Use

- The user wants to upload a product feed to Meta.
- New products or images need to be pushed to the Meta catalog.
- The user says "sync feed", "upload feed to Meta", or "update the catalog".
- A supplementary feed needs to be refreshed after image or price changes.
- The user wants to check the status of a feed upload.

## Instructions

1. **Understand the feed architecture.** Meta catalogs use Shopify **variant IDs** (not product IDs) as the `retailer_id`. All variants of a product share the same gallery images. The primary feed comes from Shopify's native sync; supplementary feeds add extra images and data.

2. **Generate or locate the feed file.** The feed should be a CSV or TSV with these columns:
   ```csv
   id,image_link,additional_image_link
   {variant_id},{main_image_url},"{img2_url},{img3_url},{img4_url}"
   ```
   - `id` = Shopify variant ID (the `retailer_id` in Meta)
   - `image_link` = primary offer image URL (Shopify CDN)
   - `additional_image_link` = comma-separated additional image URLs, wrapped in quotes

3. **Filter to synced products only.** Before uploading, filter the feed to include only products that are already fully synced in Meta (have titles, prices, etc.). Products missing from Meta's primary sync will cause "Title missing" errors. Use the feed registry or query Meta's catalog to verify:
   ```
   GET /{catalog_id}/products?filter={"retailer_id":{"is_any":["{variant_id}"]}}
   ```

4. **Upload via Data Source API.** For supplementary feeds, use the feed upload endpoint:
   ```
   POST /{catalog_id}/product_feeds
   ```
   Or update an existing data source:
   ```
   POST /{feed_id}/uploads
   Content-Type: multipart/form-data
   file: @feed-file.csv
   ```

5. **For direct item updates** (smaller batches), use the Catalog Batch API:
   ```
   POST /{catalog_id}/items_batch
   {
     "requests": [
       {
         "method": "UPDATE",
         "retailer_id": "{variant_id}",
         "data": {
           "image_link": "https://...",
           "additional_image_link": "https://...,https://..."
         }
       }
     ]
   }
   ```
   Process in batches of up to 5,000 items per request.

6. **Verify upload status.** After uploading, check the ingestion status:
   ```
   GET /{upload_session_id}?fields=end_time,num_persisted_items,num_detected_items,errors
   ```
   Note: `num_persisted_items: 0` is **normal** for supplementary feeds — it means items were matched and updated, not that the upload failed.

7. **Handle errors.** Common issues:
   - "Title missing" — product not synced from primary feed; exclude from supplementary feed
   - Rate limit (error 17) — implement exponential backoff
   - Token expired (error 190) — prompt user to refresh the access token

## Required Environment Variables

- `META_ACCESS_TOKEN` — long-lived User Access Token for the Meta Marketing API
- `META_CATALOG_ID` — the catalog ID (e.g., `528951945420575`)
- `META_FEED_ID` — the supplementary feed/data source ID (e.g., `1224589182977911`)
- `META_API_VERSION` — API version (default: `v24.0`)

## Key Files

- `data/feeds/meta-supplementary-feed-synced.csv` — the filtered feed file
- `data/active/feed-registry.json` — tracks which products have been processed
- `data/active/gallery-upload-map.json` — maps image filenames to CDN URLs

## Example Usage

```
User: Upload the latest supplementary feed to Meta
```

The skill will locate the feed file, verify it is filtered to synced products, upload via the Data Source API, and report the ingestion status.

```
User: Push image updates for these 20 products to the Meta catalog
```

The skill will use the Catalog Batch API to update the specified products directly.
