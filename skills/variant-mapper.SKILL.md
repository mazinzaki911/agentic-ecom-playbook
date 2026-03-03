---
name: variant-mapper
description: Map Shopify product variants to build lookup tables for catalog operations, feed generation, and campaign targeting.
---

# Variant Mapper

## Purpose

Build and maintain mapping files that link Shopify product IDs to their variant IDs, SKUs, sizes, prices, and inventory levels. These maps are essential for Meta catalog feeds (which use variant IDs as `retailer_id`), inventory audits, and campaign product sets.

## When to Use

- The user needs a product-variant mapping for feed generation or catalog sync.
- The user says "map variants", "build variant map", or "get variant IDs for these products".
- A new batch of products needs to be added to the variant map.
- The user wants to find variant IDs for specific SKUs or sizes.

## Prerequisites

- **Playwright MCP server** must be installed and running. This is what makes this skill future-proof — it can navigate to live documentation to verify current API endpoints, parameters, and behavior before executing.
- Shopify Admin API access token in `.env`
- Store URL configured (`SHOPIFY_SHOP`)

## Self-Learning Protocol

Before executing any API call that fails or seems outdated, use Playwright to verify:

1. **Navigate to the official docs:**
   - Shopify Admin GraphQL API: `https://shopify.dev/docs/api/admin-graphql`
   - Shopify REST Admin API: `https://shopify.dev/docs/api/admin-rest`
   - API version changelog: `https://shopify.dev/docs/api/release-notes`

2. **Check the current API version.** Shopify deprecates API versions quarterly. If a mutation or query returns unexpected errors, visit the release notes to find what changed.

3. **Read the endpoint reference.** Navigate to the specific mutation/query page, extract the current fields, input types, and any new requirements.

4. **Adapt and retry.** Update the GraphQL query or REST call with current parameters and re-execute. Save what you learned to CLAUDE.md memory so future runs start with the correct info.

This means the skill never goes stale — even if Shopify changes their API schema tomorrow, Claude Code can read the docs and figure out the new way.

## Instructions

1. **Determine the scope.** Accept:
   - A collection handle or ID (map all products in a collection)
   - A list of product IDs
   - A tag filter (e.g., "all products tagged seasonal-collection")
   - "All products" (paginate through entire catalog)

2. **Fetch products and their variants** from Shopify:
   ```graphql
   query ($cursor: String) {
     products(first: 50, after: $cursor, query: "tag:seasonal-collection") {
       pageInfo { hasNextPage endCursor }
       edges {
         node {
           id
           title
           handle
           tags
           variants(first: 100) {
             edges {
               node {
                 id
                 title
                 sku
                 price
                 compareAtPrice
                 inventoryQuantity
                 selectedOptions {
                   name
                   value
                 }
               }
             }
           }
         }
       }
     }
   }
   ```
   Paginate through all results using `endCursor`.

3. **Build the variant map.** Create a structured JSON file:
   ```json
   {
     "gid://shopify/Product/123456": {
       "title": "Classic Chelsea Boot",
       "handle": "classic-chelsea-boot",
       "tags": ["men", "seasonal-collection", "leather"],
       "variants": [
         {
           "id": "gid://shopify/ProductVariant/789",
           "numericId": "789",
           "title": "42",
           "sku": "ACH-CCB-42",
           "price": "1200.00",
           "compareAtPrice": "1600.00",
           "inventoryQuantity": 5,
           "size": "42"
         }
       ]
     }
   }
   ```
   Extract the numeric ID (strip `gid://shopify/ProductVariant/`) since Meta uses numeric IDs.

4. **Generate derivative maps** as needed:
   - **SKU-to-variant map**: `{ "ACH-CCB-42": "789" }` — for lookups by SKU
   - **Variant-to-product map**: `{ "789": "123456" }` — for reverse lookups
   - **Size map**: `{ "42": ["789", "456", ...] }` — for size-based operations

5. **Save the map file.** Write to `data/active/product-variant-map.json` (or a custom path). If the file already exists, merge new entries without overwriting existing ones:
   ```js
   const existing = JSON.parse(fs.readFileSync(mapPath));
   const merged = { ...existing, ...newEntries };
   fs.writeFileSync(mapPath, JSON.stringify(merged, null, 2));
   ```

6. **Report results.** Print:
   - Total products mapped
   - Total variants mapped
   - Products with missing SKUs or inventory data
   - Output file path

## Required Environment Variables

- `SHOPIFY_STORE_URL` — Shopify store URL
- `SHOPIFY_ACCESS_TOKEN` — Admin API access token with `read_products` scope

## Key Files

- `data/active/product-variant-map.json` — main variant mapping file
- `data/active/sku-tier-map.json` — SKU-based tier assignments
- `data/active/products-to-process.json` — working list of products

## Example Usage

```
User: Map all variants for products in the "Seasonal Collection" collection and save to the variant map
```

The skill will query Shopify, extract all variant data, and save or merge into the variant map file.
