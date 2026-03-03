---
name: catalog-image-gen
description: Generate catalog gallery images with offer overlays (e.g., "30% OFF") composited onto product photos using Sharp or node-canvas.
---

# Catalog Image Gen

## Purpose

Automate the creation of promotional gallery images for product catalog feeds. This skill takes source product images, composites a branded offer overlay (discount badge, sale banner, price callout), and outputs web-optimized JPEGs ready for upload to a CDN or Meta catalog.

## When to Use

- The user asks to generate offer images, sale images, or promotional catalog images.
- A new batch of products needs gallery images before being added to a Meta supplementary feed.
- The user wants to refresh existing offer overlays with a new discount percentage or design.
- The user says "generate images for these products" in the context of catalog or feed work.

## Prerequisites

- **Playwright MCP server** must be installed and running. This is what makes this skill future-proof — it can navigate to live platform documentation to verify current API endpoints, parameters, and behavior before executing.
- Meta Marketing API access token in `.env`
- Shopify Admin API access token in `.env`
- Store URL and Ad Account ID configured

## Self-Learning Protocol

Before executing any API call that fails or seems outdated, use Playwright to verify:

1. **Navigate to the official docs:**
   - Meta Marketing API: `https://developers.facebook.com/docs/marketing-api/reference/`
   - Shopify Admin GraphQL API: `https://shopify.dev/docs/api/admin-graphql`
   - Both platforms' changelogs for breaking changes

2. **Check the current API version.** Both Meta and Shopify deprecate versions regularly. If a call fails, visit the changelog to find what changed.

3. **Read the endpoint reference.** Navigate to the specific endpoint page, extract the current parameters, required fields, and any new restrictions.

4. **Adapt and retry.** Update the API call with current parameters and re-execute. Save what you learned to CLAUDE.md memory so future runs start with the correct info.

This means the skill never goes stale — even if either platform ships a breaking change tomorrow, Claude Code can read the docs and figure out the new way.

## Instructions

1. **Identify source images.** Look for a product-image-map file (typically `data/active/product-image-map.json`) that maps product IDs to their source image filenames or URLs. If none exists, ask the user for the product list.

2. **Determine overlay parameters.** Ask or infer:
   - Discount text (e.g., "30% OFF", "SALE", "BUY 1 GET 1")
   - Badge position (top-right, bottom-left, centered banner)
   - Brand colors (default: black background, white text, accent color)
   - Output dimensions (default: 1080x1080 for Meta catalog)

3. **Write the image generation script.** Create a Node.js script that:
   ```js
   const sharp = require('sharp');
   const path = require('path');
   ```
   - Reads each source image from the local filesystem or downloads from a URL.
   - Resizes to the target dimensions (1080x1080) with `sharp.resize({ fit: 'cover' })`.
   - Creates an SVG overlay with the discount text, positioned according to the badge config.
   - Composites the overlay onto the product image using `sharp.composite()`.
   - Outputs to an `output/gallery/` directory with a naming convention like `{product_id}_offer.jpg`.
   - Uses JPEG quality 85 for web optimization.

4. **Handle batch processing.** Process images in batches of 10 with concurrency control to avoid memory issues:
   ```js
   const pLimit = require('p-limit');
   const limit = pLimit(10);
   ```

5. **Update the product-image-map.** After generation, update the map file to record which products now have gallery images and their output paths.

6. **Report results.** Print a summary: total processed, successful, failed, and output directory path.

## Required Environment Variables

- None required for image generation itself.
- `SHOPIFY_STORE_URL` — needed only if downloading source images from Shopify CDN.

## Key Files

- `data/active/product-image-map.json` — maps product IDs to source images
- `data/active/products-to-process.json` — list of products needing images
- `output/gallery/` — default output directory for generated images

## Example Usage

```
User: Generate offer images for the 47 products in products-to-process.json with a "30% OFF" badge
```

The skill will read the product list, load source images, composite the overlay, and save optimized JPEGs to the output directory.

## Dependencies

- `sharp` (image processing)
- `p-limit` (concurrency control)
- Optionally `@napi-rs/canvas` for complex text rendering
