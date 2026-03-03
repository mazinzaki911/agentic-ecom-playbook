---
name: metafield-writer
description: Set custom metafields on Shopify products, variants, and collections using the GraphQL Admin API for storefront display, filtering, and app integrations.
---

# Metafield Writer

## Purpose

Create and update metafields on Shopify resources (products, variants, collections, shop). Metafields store custom data used by theme sections, storefront filtering, app integrations, and headless commerce. This skill handles metafield definitions, value types, and bulk operations.

## When to Use

- The user wants to set a custom field on a product or collection.
- The user says "set metafield", "add custom data", or "update metafield".
- Storefront filtering requires metafield values (e.g., material, gender, season).
- A theme section reads from a metafield that needs to be populated.
- Metafield definitions need to be created before setting values.

## Instructions

1. **Check if the metafield definition exists.** Before setting values, verify the definition:
   ```graphql
   query {
     metafieldDefinitions(
       first: 10,
       ownerType: PRODUCT,
       query: "namespace:custom AND key:season"
     ) {
       edges {
         node {
           id
           name
           namespace
           key
           type { name }
           pinnedPosition
         }
       }
     }
   }
   ```

2. **Create the definition if needed.** Metafield definitions control the data type and validation:
   ```graphql
   mutation {
     metafieldDefinitionCreate(definition: {
       name: "Season",
       namespace: "custom",
       key: "season",
       type: "single_line_text_field",
       ownerType: PRODUCT,
       pin: true,
       visibleToStorefrontApi: true,
       validations: [
         { name: "choices", value: "[\"Spring\",\"Summer\",\"Fall\",\"Winter\"]" }
       ]
     }) {
       createdDefinition { id }
       userErrors { field message }
     }
   }
   ```
   Common types: `single_line_text_field`, `number_integer`, `number_decimal`, `boolean`, `json`, `list.single_line_text_field`, `rich_text_field`, `url`, `color`.

3. **Set metafield values on products.**
   ```graphql
   mutation {
     productUpdate(input: {
       id: "gid://shopify/Product/{id}",
       metafields: [
         {
           namespace: "custom",
           key: "season",
           value: "Winter",
           type: "single_line_text_field"
         },
         {
           namespace: "custom",
           key: "material",
           value: "Genuine Leather",
           type: "single_line_text_field"
         }
       ]
     }) {
       product { id title }
       userErrors { field message }
     }
   }
   ```

4. **Set metafield values on collections.**
   ```graphql
   mutation {
     collectionUpdate(input: {
       id: "gid://shopify/Collection/{id}",
       metafields: [
         {
           namespace: "custom",
           key: "filter_enabled",
           value: "true",
           type: "boolean"
         },
         {
           namespace: "custom",
           key: "banner_text",
           value: "Up to 50% Off Winter Collection",
           type: "single_line_text_field"
         }
       ]
     }) {
       collection { id title }
       userErrors { field message }
     }
   }
   ```

5. **Bulk set metafields** using the `metafieldsSet` mutation (more efficient for many resources):
   ```graphql
   mutation {
     metafieldsSet(metafields: [
       {
         ownerId: "gid://shopify/Product/111",
         namespace: "custom",
         key: "season",
         value: "Winter",
         type: "single_line_text_field"
       },
       {
         ownerId: "gid://shopify/Product/222",
         namespace: "custom",
         key: "season",
         value: "Winter",
         type: "single_line_text_field"
       }
     ]) {
       metafields { id namespace key value }
       userErrors { field message }
     }
   }
   ```
   Up to 25 metafields per mutation call.

6. **Enable storefront filtering.** If the metafield should appear as a filter on collection pages, ensure:
   - The definition has `visibleToStorefrontApi: true`
   - The definition is pinned (`pin: true`)
   - The Online Store has the filter enabled in its settings

7. **Verify metafield values.** After setting, query to confirm:
   ```graphql
   query {
     product(id: "gid://shopify/Product/{id}") {
       metafield(namespace: "custom", key: "season") {
         value
         type
       }
     }
   }
   ```

8. **Delete metafield values** if needed:
   ```graphql
   mutation {
     metafieldDelete(input: { id: "gid://shopify/Metafield/{metafield_id}" }) {
       deletedId
       userErrors { field message }
     }
   }
   ```

## Required Environment Variables

- `SHOPIFY_STORE_URL` — Shopify store URL
- `SHOPIFY_ACCESS_TOKEN` — Admin API access token with `write_products`, `write_metafields` scopes

## Common Metafield Patterns

| Use Case | Namespace | Key | Type |
|----------|-----------|-----|------|
| Season filter | custom | season | single_line_text_field |
| Material filter | custom | material | single_line_text_field |
| Gender filter | custom | gender | single_line_text_field |
| Sale badge text | custom | badge_text | single_line_text_field |
| Featured flag | custom | featured | boolean |
| Sort priority | custom | sort_order | number_integer |
| Collection banner | custom | banner_text | single_line_text_field |

## Example Usage

```
User: Set the "season" metafield to "Winter" on all products in the winter boots collection
```

The skill will check for the definition, create it if needed, fetch all products in the collection, and set the metafield value on each.
