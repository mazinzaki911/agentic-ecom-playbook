---
name: product-tagger
description: Bulk add or remove tags on Shopify products via the GraphQL Admin API, with support for collection-based, query-based, and file-based product selection.
---

# Product Tagger

## Purpose

Add or remove tags on Shopify products in bulk. Tags drive collection membership (automated collections), campaign targeting, feed filtering, and storefront display logic. This skill handles tagging hundreds of products efficiently using GraphQL bulk operations.

## When to Use

- The user wants to add or remove tags on products.
- The user says "tag products", "add tag", "remove tag", or "untag".
- A campaign or sale requires products to be tagged (e.g., "SALEJAN", "winter-sale", "bogo").
- Tags need to be cleaned up after a promotion ends.
- Products in a collection need a common tag applied.

## Instructions

1. **Identify the target products.** Accept:
   - A collection handle or ID (tag all products in a collection)
   - A search query (e.g., `tag:winter AND product_type:boots`)
   - A JSON file with product IDs
   - Specific product IDs or handles

2. **Determine the operation:**
   - **Add tags**: Append new tags without removing existing ones
   - **Remove tags**: Remove specific tags, preserving others
   - **Replace tags**: Replace the entire tag list (use with caution)

3. **Fetch current products and their tags:**
   ```graphql
   query ($cursor: String) {
     products(first: 50, after: $cursor, query: "{user_query}") {
       pageInfo { hasNextPage endCursor }
       edges {
         node {
           id
           title
           tags
         }
       }
     }
   }
   ```
   Paginate to collect all matching products.

4. **Calculate new tag lists.** For each product:
   ```js
   // Adding tags
   const newTags = [...new Set([...existingTags, ...tagsToAdd])];

   // Removing tags
   const newTags = existingTags.filter(t => !tagsToRemove.includes(t));
   ```

5. **Apply tags using tagsAdd/tagsRemove mutations** (preferred for add/remove):
   ```graphql
   mutation {
     tagsAdd(id: "gid://shopify/Product/{id}", tags: ["winter-sale", "30-off"]) {
       node { ... on Product { id tags } }
       userErrors { field message }
     }
   }
   ```
   ```graphql
   mutation {
     tagsRemove(id: "gid://shopify/Product/{id}", tags: ["SALEJAN"]) {
       node { ... on Product { id tags } }
       userErrors { field message }
     }
   }
   ```
   These mutations are atomic — `tagsAdd` won't duplicate existing tags, and `tagsRemove` is safe if the tag doesn't exist.

6. **Batch for efficiency.** Process multiple products per request:
   ```graphql
   mutation {
     p1: tagsAdd(id: "gid://shopify/Product/111", tags: ["sale"]) {
       userErrors { message }
     }
     p2: tagsAdd(id: "gid://shopify/Product/222", tags: ["sale"]) {
       userErrors { message }
     }
     p3: tagsAdd(id: "gid://shopify/Product/333", tags: ["sale"]) {
       userErrors { message }
     }
   }
   ```
   Batch up to 10 products per mutation. Add a small delay between batches (500ms) to respect rate limits.

7. **For very large operations (500+ products)**, use Shopify's bulk operation API:
   ```graphql
   mutation {
     bulkOperationRunMutation(
       mutation: "mutation($input: ProductInput!) { productUpdate(input: $input) { product { id tags } userErrors { field message } } }",
       stagedUploadPath: "{staged_upload_path}"
     ) {
       bulkOperation { id status }
       userErrors { message }
     }
   }
   ```
   Upload a JSONL file with one product per line.

8. **Report results.**
   ```
   Tagging Summary
   ===============
   Operation: ADD tags ["winter-sale", "30-off"]
   Products processed: 147
   Successful: 145
   Errors: 2 (Product X: permission denied, Product Y: not found)
   ```

## Required Environment Variables

- `SHOPIFY_STORE_URL` — Shopify store URL
- `SHOPIFY_ACCESS_TOKEN` — Admin API access token with `write_products` scope

## Example Usage

```
User: Add the "end-of-season" tag to all products in the Winter Boots collection
```

The skill will fetch all products in the collection, add the tag to each, and report results.

```
User: Remove the "SALEJAN" tag from all products
```

The skill will find all products with the SALEJAN tag and remove it in bulk.
