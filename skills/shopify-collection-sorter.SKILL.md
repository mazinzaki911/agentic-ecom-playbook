---
name: shopify-collection-sorter
description: Sort Shopify collections by sales velocity, stock depth, or custom criteria using the Shopify Admin GraphQL API.
---

# Shopify Collection Sorter

## Purpose

Programmatically reorder products within Shopify collections based on data-driven criteria: best-sellers first, highest stock first, newest arrivals, or custom weighted scores. Manual collection sorting is tedious and this skill automates it at scale across multiple collections.

## When to Use

- The user wants to sort a collection by best-sellers, stock levels, or another metric.
- The user says "sort collection", "reorder products", or "put best sellers first".
- Multiple collections need to be sorted in bulk (e.g., all seasonal collections).
- The user wants to push high-stock items to the top to accelerate sell-through.

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

1. **Identify target collections.** Ask the user which collections to sort, or accept collection IDs/handles. To find collections:
   ```graphql
   query {
     collections(first: 50, query: "title:*winter*") {
       edges {
         node {
           id
           title
           handle
           productsCount
         }
       }
     }
   }
   ```

2. **Determine the sort strategy.** Options:
   - **Best-sellers**: Sort by total units sold in the last 30/60/90 days (requires sales data from Shopify Analytics API or order history)
   - **Stock depth**: Sort by total inventory quantity (highest first to push slow movers)
   - **Price high-to-low / low-to-high**: Sort by variant price
   - **Custom score**: Weighted combination (e.g., 70% sales velocity + 30% stock depth)

3. **Fetch product data for the collection.**
   ```graphql
   query {
     collection(id: "gid://shopify/Collection/{id}") {
       products(first: 250, sortKey: COLLECTION_DEFAULT) {
         edges {
           node {
             id
             title
             totalInventory
             variants(first: 100) {
               edges {
                 node {
                   id
                   inventoryQuantity
                   price
                 }
               }
             }
           }
         }
       }
     }
   }
   ```
   Handle pagination for collections with 250+ products.

4. **Fetch sales data** (for best-seller sort). Query recent orders:
   ```graphql
   query {
     orders(first: 250, query: "created_at:>2025-12-01", sortKey: CREATED_AT) {
       edges {
         node {
           lineItems(first: 50) {
             edges {
               node {
                 product { id }
                 quantity
               }
             }
           }
         }
       }
     }
   }
   ```
   Aggregate units sold per product ID.

5. **Compute sort order.** Rank products by the chosen criteria and build an ordered list of product IDs.

6. **Apply the new sort order** using the `collectionReorderProducts` mutation:
   ```graphql
   mutation {
     collectionReorderProducts(
       id: "gid://shopify/Collection/{id}",
       moves: [
         { id: "gid://shopify/Product/{pid}", newPosition: "0" },
         { id: "gid://shopify/Product/{pid}", newPosition: "1" },
         ...
       ]
     ) {
       job { id done }
       userErrors { field message }
     }
   }
   ```
   The `moves` array sets each product's new position. Process in batches if the collection is large.

7. **Verify the result.** Re-fetch the collection to confirm the new order. Print the top 10 products in the new sort order.

## Required Environment Variables

- `SHOPIFY_STORE_URL` — Shopify store URL (e.g., `your-store.myshopify.com`)
- `SHOPIFY_ACCESS_TOKEN` — Admin API access token with `read_products`, `write_products`, `read_orders` scopes

## Example Usage

```
User: Sort the "Men's Leather Shoes" collection by stock depth, highest first
```

The skill will fetch all products in the collection, rank by total inventory, and reorder via the GraphQL mutation.
