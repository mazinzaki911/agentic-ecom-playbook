---
name: collections-mapper
description: Map and audit Shopify collections, building a registry with rules, product counts, and sync status to Meta product sets.
---

# Collections Mapper

## Purpose

Build and maintain a comprehensive registry of all Shopify collections, their automation rules, product counts, and corresponding Meta product sets. This skill bridges the gap between Shopify's collection system and Meta's product set system, ensuring products are correctly organized for both the storefront and advertising campaigns.

## When to Use

- The user says "map collections", "audit collections", "check collection sync", or "build collection registry".
- The user runs `/collections-map`, `/collections-build`, or `/collections-audit`.
- New collections have been created and need to be matched to Meta product sets.
- The user needs a full picture of how collections map to ad campaigns.

## Prerequisites

- **Playwright MCP server** must be installed and running. This is used to verify Shopify and Meta documentation for collection/product set APIs.
- Shopify Admin API access token in `.env`
- Meta Marketing API access token in `.env` (for product set sync checking)
- Catalog ID configured

## Self-Learning Protocol

Before executing any API call that fails or seems outdated, use Playwright to verify:

1. **Navigate to the official docs:**
   - Shopify Collections API: `https://shopify.dev/docs/api/admin-graphql/latest/queries/collections`
   - Meta Product Sets: `https://developers.facebook.com/docs/marketing-api/reference/product-set/`
   - Meta Catalog API: `https://developers.facebook.com/docs/marketing-api/reference/product-catalog/product-sets/`

2. **Check for schema changes** in both Shopify collection fields and Meta product set filter syntax.

3. **Adapt queries** to match current API specifications.

## Instructions

### `/collections-map` — Map All Collections

1. **Fetch all Shopify collections** (smart + custom):
   ```graphql
   query {
     collections(first: 100) {
       edges {
         node {
           id
           title
           handle
           productsCount { count }
           ruleSet {
             appliedDisjunctively
             rules { column relation condition }
           }
           sortOrder
           updatedAt
         }
       }
       pageInfo { hasNextPage endCursor }
     }
   }
   ```
   Paginate through all collections.

2. **Fetch all Meta product sets:**
   ```
   GET /{catalog_id}/product_sets?fields=id,name,filter,product_count
   ```

3. **Build the mapping registry:**
   ```json
   {
     "collections": [
       {
         "shopify_id": "gid://shopify/Collection/123",
         "title": "Winter Arrivals",
         "handle": "winter-arrivals",
         "product_count": 85,
         "rule_type": "smart",
         "rules": [{"column": "TAG", "relation": "EQUALS", "condition": "winter"}],
         "meta_product_set_id": "456789",
         "meta_product_set_name": "Winter Arrivals",
         "meta_product_count": 82,
         "sync_status": "partial",
         "count_diff": 3
       }
     ]
   }
   ```

4. **Save the registry** to `data/active/collection-registry.json`.

### `/collections-build` — Build Missing Product Sets

1. **Identify collections without Meta product sets.**
2. **For each unmapped collection**, create a corresponding Meta product set:
   ```
   POST /{catalog_id}/product_sets
   {
     "name": "{collection_title}",
     "filter": {"retailer_id": {"is_any": [{variant_ids}]}}
   }
   ```
3. **Update the registry** with the new mapping.

### `/collections-audit` — Audit Sync Status

1. **Load the registry** (or build it fresh with `/collections-map`).
2. **For each mapping, check:**
   - Product count matches between Shopify and Meta
   - All Shopify products in the collection have variant IDs in the Meta product set
   - No stale product sets (Meta sets for deleted Shopify collections)
3. **Report findings:**
   ```
   Collection Sync Audit
   =====================
   Total Collections: 45
   Mapped to Meta: 38
   Unmapped: 7
   Fully Synced: 32
   Partial Sync: 6 (count mismatches)
   Stale Meta Sets: 2

   Action Items:
   - Create Meta product sets for 7 unmapped collections
   - Investigate 6 count mismatches
   - Delete 2 stale Meta product sets
   ```

## Required Environment Variables

- `SHOPIFY_STORE_URL` — Shopify store URL
- `SHOPIFY_ACCESS_TOKEN` — Admin API access token
- `META_ACCESS_TOKEN` — Meta access token
- `META_CATALOG_ID` — product catalog ID

## Example Usage

```
User: /collections-map
```

Builds a full registry of all Shopify collections mapped to Meta product sets.

```
User: /collections-audit
```

Checks all mappings for sync issues and reports findings.

```
User: /collections-build for collections tagged "new-season"
```

Creates Meta product sets for any unmapped collections matching the filter.
