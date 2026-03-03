---
name: price-engine
description: Apply percentage discounts, set compare-at prices, and manage sale pricing across Shopify product variants via GraphQL.
---

# Price Engine

## Purpose

Automate bulk pricing operations on Shopify: apply percentage discounts to variants, set or clear compare-at prices, round prices to the nearest 5 or 10, and restore original pricing after a sale ends. Handles the math, the API calls, and the audit trail.

## When to Use

- The user wants to apply a discount (e.g., "30% off winter collection").
- The user says "set sale prices", "update compare-at price", or "apply discount".
- Prices need to be rounded after discount calculation.
- The user wants to revert products to their original prices after a promotion.
- A BOGO or end-of-season pricing change is needed.

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

1. **Identify target products.** Accept:
   - A collection handle or ID
   - A list of product IDs or variant IDs
   - A tag-based filter (e.g., all products tagged "SALEJAN")
   - A JSON file with product data (e.g., `data/active/end-of-season-products.json`)

2. **Determine the pricing operation:**
   - **Apply discount**: Set `price = compare_at_price * (1 - discount_rate)`. The `compare_at_price` becomes the original price (strikethrough in storefront).
   - **Set compare-at**: Set `compare_at_price` to the current price, then reduce `price`.
   - **Clear compare-at**: Remove `compare_at_price` (set to null) to end a sale.
   - **Round prices**: Round to nearest 5 or 10 EGP (e.g., 742 -> 740, 748 -> 750).

3. **Fetch current prices** for all target variants:
   ```graphql
   query {
     product(id: "gid://shopify/Product/{id}") {
       title
       variants(first: 100) {
         edges {
           node {
             id
             price
             compareAtPrice
             inventoryQuantity
           }
         }
       }
     }
   }
   ```

4. **Calculate new prices.** For each variant:
   ```js
   const originalPrice = parseFloat(variant.compareAtPrice || variant.price);
   const discountedPrice = originalPrice * (1 - discountRate);
   const roundedPrice = Math.round(discountedPrice / roundTo) * roundTo;
   ```
   Always preserve the original price in `compare_at_price` before discounting.

5. **Create a backup** before applying changes. Save current prices to a timestamped JSON file:
   ```json
   {
     "timestamp": "2026-03-03T12:00:00Z",
     "operation": "30% discount",
     "variants": [
       {"id": "gid://...", "originalPrice": "1200.00", "originalCompareAt": null}
     ]
   }
   ```

6. **Apply the price changes** using the `productVariantsBulkUpdate` mutation:
   ```graphql
   mutation {
     productVariantsBulkUpdate(
       productId: "gid://shopify/Product/{pid}",
       variants: [
         {
           id: "gid://shopify/ProductVariant/{vid}",
           price: "{new_price}",
           compareAtPrice: "{compare_at_price}"
         }
       ]
     ) {
       productVariants { id price compareAtPrice }
       userErrors { field message }
     }
   }
   ```
   Process one product at a time (all its variants in one mutation). Add a delay between products to respect rate limits (2 requests/second for GraphQL).

7. **Report results.** Print a summary table:
   - Total variants updated
   - Average discount applied
   - Price range (min/max)
   - Any errors encountered
   - Path to the backup file

## Required Environment Variables

- `SHOPIFY_STORE_URL` — Shopify store URL
- `SHOPIFY_ACCESS_TOKEN` — Admin API access token with `write_products` scope

## Key Concepts

- **compare_at_price** is the "was" price shown with a strikethrough. It must be higher than `price` for the sale badge to appear.
- Setting `compare_at_price` to `null` or `""` clears it (no strikethrough).
- Shopify prices are strings (e.g., `"1200.00"`), not numbers.
- Always backup before bulk price changes — mistakes are costly.

## Example Usage

```
User: Apply 25% off to all products tagged "winter-sale" and round to nearest 10 EGP
```

The skill will fetch all winter-sale products, calculate discounted prices, round them, backup originals, and apply via GraphQL.
