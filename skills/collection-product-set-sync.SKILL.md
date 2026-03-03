---
name: collection-product-set-sync
description: Sync Shopify collections to Meta product sets for DPA targeting, ensuring catalog segments match storefront collections.
---

# Collection Product Set Sync

## Purpose

Create and maintain Meta product sets that mirror Shopify collections. Product sets are subsets of a Meta catalog used to target specific product groups in DPA campaigns, collection ads, and Advantage+ catalog ads. This skill ensures that when a Shopify collection changes, the corresponding Meta product set stays in sync.

## When to Use

- The user creates a new Shopify collection and needs a matching Meta product set.
- The user says "sync collection to Meta", "create product set", or "update product set".
- A DPA campaign needs to target products from a specific Shopify collection.
- Product sets are out of sync with their Shopify collections (products added/removed).
- The user wants to audit which collections have corresponding product sets.

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

1. **Identify the Shopify collection.** Fetch the collection and its products:
   ```graphql
   query {
     collectionByHandle(handle: "{collection_handle}") {
       id
       title
       productsCount
       products(first: 250) {
         pageInfo { hasNextPage endCursor }
         edges {
           node {
             id
             title
             variants(first: 100) {
               edges {
                 node {
                   id
                 }
               }
             }
           }
         }
       }
     }
   }
   ```
   Paginate to get all products. Extract all variant IDs (numeric form) since Meta uses variant IDs as `retailer_id`.

2. **Check for an existing product set.** List product sets in the catalog:
   ```
   GET /{catalog_id}/product_sets?fields=name,id,filter,product_count
   ```
   Look for a product set matching the collection name or handle.

3. **Create a new product set** (if none exists). There are two approaches:

   **Option A: Filter-based** (dynamic, auto-updates when products match the filter):
   ```
   POST /{catalog_id}/product_sets
   {
     "name": "{collection_name}",
     "filter": {
       "retailer_id": {
         "is_any": ["{variant_id_1}", "{variant_id_2}", "..."]
       }
     }
   }
   ```
   Note: Filter by `retailer_id` is explicit (list of IDs). For dynamic matching, use product attributes:
   ```json
   {
     "filter": {
       "product_type": {"i_contains": "boots"},
       "custom_label_0": {"eq": "winter-sale"}
     }
   }
   ```

   **Option B: Rule-based using custom labels** (if the feed includes custom labels):
   ```json
   {
     "filter": {
       "custom_label_0": {"eq": "{collection_handle}"}
     }
   }
   ```
   This requires the supplementary feed to include `custom_label_0` with the collection handle for each product.

4. **Update an existing product set** when products change:
   ```
   POST /{product_set_id}
   {
     "filter": {
       "retailer_id": {
         "is_any": ["{updated_variant_id_list}"]
       }
     }
   }
   ```

5. **Verify the product set.** Check that the product count matches:
   ```
   GET /{product_set_id}?fields=name,product_count,filter
   ```
   Compare `product_count` with the Shopify collection's product count. Note that variant-level counting may differ from product-level counting.

6. **Bulk sync multiple collections.** If syncing many collections:
   - Fetch all collections from Shopify
   - Fetch all product sets from Meta
   - Match by name or a custom naming convention (e.g., `shopify_{handle}`)
   - Create missing product sets
   - Update out-of-sync product sets
   - Report orphaned product sets (Meta sets with no matching Shopify collection)

7. **Handle large product sets.** Meta product set filters have a limit on the number of `retailer_id` values. For collections with 500+ products:
   - Use attribute-based filters instead of explicit ID lists
   - Ensure the supplementary feed includes `custom_label_0` through `custom_label_4` with relevant collection tags
   - Update the feed first, then create the product set with a `custom_label` filter

8. **Report results.** For each collection synced:
   ```
   Collection-to-Product-Set Sync Report
   ======================================
   Collection                  | Product Set ID     | Products | Status
   Men's Winter Boots          | 12345678           | 23       | SYNCED
   Women's Winter Boots        | 12345679           | 19       | SYNCED
   Spring Loafers              | (new) 12345680     | 15       | CREATED
   Summer Sandals              | 12345681           | 12       | UPDATED (+3 products)
   ```

## Required Environment Variables

- `META_ACCESS_TOKEN` — long-lived User Access Token
- `META_CATALOG_ID` — product catalog ID (e.g., `528951945420575`)
- `SHOPIFY_STORE_URL` — Shopify store URL
- `SHOPIFY_ACCESS_TOKEN` — Admin API access token with `read_products` scope

## Key Concepts

- **Product Set** = a filtered subset of a Meta catalog, used for ad targeting.
- **retailer_id** = Shopify variant ID (numeric), the primary key linking Shopify to Meta.
- Product sets can use static filters (`retailer_id` list) or dynamic filters (product attributes like `product_type`, `custom_label_*`).
- Dynamic filters auto-update when products matching the criteria are added/removed from the catalog.
- Static filters (explicit ID lists) need manual updating when the collection changes.

## Example Usage

```
User: Create a Meta product set for the "Women's End of Season" collection
```

The skill will fetch the collection from Shopify, extract all variant IDs, create a product set in Meta, and verify the product count.

```
User: Sync all seasonal collections to Meta product sets
```

The skill will fetch all seasonal collections, compare with existing product sets, create/update as needed, and report the sync status.
