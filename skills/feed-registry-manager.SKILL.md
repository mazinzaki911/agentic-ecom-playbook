---
name: feed-registry-manager
description: Track which products have been processed through the feed pipeline — image generation, CDN upload, feed inclusion, and Meta sync status.
---

# Feed Registry Manager

## Purpose

Maintain a central registry that tracks the processing status of every product in the catalog feed pipeline. Each product moves through stages: image generated, uploaded to CDN, included in feed file, uploaded to Meta. The registry prevents duplicate processing, identifies gaps, and provides audit trails.

## When to Use

- The user asks "which products still need images?" or "what's missing from the feed?"
- Before running any feed pipeline step, to determine what needs processing.
- The user says "audit the feed", "check feed status", or "update the registry".
- After a pipeline step completes, to record what was processed.
- The user wants to see a summary of feed coverage.

## Instructions

1. **Load or initialize the registry.** The registry lives at `data/active/feed-registry.json`:
   ```json
   {
     "lastUpdated": "2026-03-03T10:00:00Z",
     "products": {
       "gid://shopify/Product/123456": {
         "title": "Classic Chelsea Boot",
         "variantCount": 5,
         "stages": {
           "imageGenerated": { "status": true, "timestamp": "2026-02-15T08:00:00Z", "files": ["123456_offer.jpg"] },
           "cdnUploaded": { "status": true, "timestamp": "2026-02-15T09:00:00Z", "urls": ["https://cdn.shopify.com/..."] },
           "feedIncluded": { "status": true, "timestamp": "2026-02-15T10:00:00Z", "feedFile": "meta-supplementary-feed-synced.csv" },
           "metaSynced": { "status": true, "timestamp": "2026-02-15T11:00:00Z", "catalogId": "528951945420575" }
         },
         "tags": ["winter-boots", "men"]
       }
     }
   }
   ```

2. **Query the registry.** Support these queries:
   - **Missing stage**: Find all products where a specific stage is `false` or missing
   - **Fully processed**: Find products where all stages are complete
   - **By tag**: Filter products by tags
   - **By date**: Find products processed before/after a date
   - **Statistics**: Count products at each stage

3. **Update after pipeline steps.** When a pipeline step completes, update the registry:
   ```js
   function updateRegistry(productId, stage, data) {
     registry.products[productId].stages[stage] = {
       status: true,
       timestamp: new Date().toISOString(),
       ...data
     };
     registry.lastUpdated = new Date().toISOString();
   }
   ```

4. **Audit mode.** Cross-reference the registry with actual files:
   - Check that image files listed in the registry actually exist on disk
   - Verify CDN URLs are accessible (HEAD request)
   - Confirm variant IDs in the feed file match the registry
   - Query Meta catalog to verify sync status:
     ```
     GET /{catalog_id}/products?filter={"retailer_id":{"is_any":["{variant_id}"]}}
     ```

5. **Generate reports.** Output a formatted summary:
   ```
   Feed Pipeline Status Report
   ===========================
   Total products: 47
   Images generated: 45 (95.7%)
   Uploaded to CDN: 43 (91.5%)
   Included in feed: 43 (91.5%)
   Synced to Meta: 41 (87.2%)

   Missing images: Product A, Product B
   Missing CDN upload: Product C, Product D, Product A, Product B
   Not in feed: Product C, Product D, Product A, Product B
   Not synced to Meta: Product E, Product F, Product C, Product D, Product A, Product B
   ```

6. **Save the registry.** Write back to `data/active/feed-registry.json` with pretty-printed JSON.

## Required Environment Variables

- `META_ACCESS_TOKEN` — for verifying Meta sync status (optional, only for audit)
- `META_CATALOG_ID` — catalog ID (optional, only for audit)

## Key Files

- `data/active/feed-registry.json` — the central registry file
- `data/active/product-image-map.json` — cross-reference for image generation status
- `data/active/gallery-upload-map.json` — cross-reference for CDN upload status
- `data/feeds/meta-supplementary-feed-synced.csv` — cross-reference for feed inclusion

## Example Usage

```
User: Which products are missing from the Meta feed?
```

The skill will load the registry, find all products where `metaSynced` is false, and report them with details.
