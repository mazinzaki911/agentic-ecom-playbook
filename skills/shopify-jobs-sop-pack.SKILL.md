---
name: shopify-jobs-sop-pack
description: Collection of 5 Shopify SOPs — product management, order operations, inventory management, pricing operations, and collection management.
---

# Shopify Jobs SOP Pack

## Purpose

Provide a complete set of Standard Operating Procedures for managing a Shopify store via the Admin API. This pack covers the core operational tasks that e-commerce teams perform daily: managing products, processing orders, tracking inventory, adjusting prices, and organizing collections.

## When to Use

- The user wants to perform Shopify store operations via API.
- The user runs `/list-products`, `/check-orders`, `/check-inventory`, `/update-prices`, or `/manage-collections`.
- Daily or weekly operational tasks need to be executed.
- Store data needs to be audited or updated in bulk.

## Prerequisites

- **Playwright MCP server** must be installed and running. This is what makes these SOPs future-proof — they can navigate to live Shopify documentation to verify current API endpoints, GraphQL schema, and field names.
- Shopify Admin API access token in `.env`
- Store URL configured

## Self-Learning Protocol

Before executing any API call that fails or seems outdated, use Playwright to verify:

1. **Navigate to the official docs:**
   - Shopify Admin GraphQL API: `https://shopify.dev/docs/api/admin-graphql`
   - Product API: `https://shopify.dev/docs/api/admin-graphql/latest/queries/products`
   - Order API: `https://shopify.dev/docs/api/admin-graphql/latest/queries/orders`
   - Inventory API: `https://shopify.dev/docs/api/admin-graphql/latest/queries/inventoryItems`
   - API Release Notes: `https://shopify.dev/docs/api/release-notes`

2. **Check the current API version.** Shopify deprecates versions quarterly. Verify the version in use is still supported.

3. **Read the query/mutation reference.** Navigate to the specific query or mutation page, extract current fields and input types.

4. **Adapt and retry.** Update the GraphQL query with current fields and re-execute. Save learnings to CLAUDE.md.

## Instructions

### SOP 1: `/list-products` — Product Management

1. **List products** with filters:
   ```graphql
   query {
     products(first: 50, query: "{filter}") {
       edges {
         node {
           id
           title
           status
           vendor
           productType
           tags
           totalInventory
           variants(first: 10) {
             edges {
               node {
                 id
                 title
                 sku
                 price
                 compareAtPrice
                 inventoryQuantity
               }
             }
           }
         }
       }
       pageInfo { hasNextPage endCursor }
     }
   }
   ```

2. **Filter options:**
   - By status: `status:active`, `status:draft`, `status:archived`
   - By tag: `tag:{tag_name}`
   - By vendor: `vendor:{vendor_name}`
   - By product type: `product_type:{type}`

3. **Format output** as a readable table with key metrics.

### SOP 2: `/check-orders` — Order Operations

1. **Fetch recent orders:**
   ```graphql
   query {
     orders(first: 20, sortKey: CREATED_AT, reverse: true, query: "{filter}") {
       edges {
         node {
           id
           name
           createdAt
           displayFinancialStatus
           displayFulfillmentStatus
           totalPriceSet { shopMoney { amount currencyCode } }
           customer { firstName lastName email }
           lineItems(first: 10) {
             edges { node { title quantity sku } }
           }
         }
       }
     }
   }
   ```

2. **Filter options:**
   - By status: `financial_status:paid`, `fulfillment_status:unfulfilled`
   - By date: `created_at:>{date}`
   - By customer: `email:{email}`

3. **Report summary:** total orders, total revenue, unfulfilled count, average order value.

### SOP 3: `/check-inventory` — Inventory Management

1. **Fetch inventory levels** for specific products or all products:
   ```graphql
   query {
     productVariants(first: 50, query: "{filter}") {
       edges {
         node {
           id
           displayName
           sku
           inventoryQuantity
           inventoryItem {
             id
             tracked
             inventoryLevels(first: 5) {
               edges {
                 node {
                   location { name }
                   quantities(names: ["available", "committed", "on_hand"]) {
                     name
                     quantity
                   }
                 }
               }
             }
           }
         }
       }
     }
   }
   ```

2. **Flag low stock** items (quantity < threshold, default 5).

3. **Flag out-of-stock** items (quantity = 0).

4. **Report inventory summary** by location.

### SOP 4: `/update-prices` — Pricing Operations

1. **Gather pricing changes:**
   - Products/variants to update
   - New price and/or compare-at price
   - Discount percentage (to calculate from compare-at)

2. **Update variant prices:**
   ```graphql
   mutation {
     productVariantsBulkUpdate(
       productId: "{product_id}",
       variants: [
         {
           id: "{variant_id}",
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

3. **Verify updates** by re-fetching the updated variants.

4. **Report:** products updated, old vs. new prices, any errors.

### SOP 5: `/manage-collections` — Collection Management

1. **List collections:**
   ```graphql
   query {
     collections(first: 50, query: "{filter}") {
       edges {
         node {
           id
           title
           handle
           productsCount { count }
           ruleSet { rules { column relation condition } }
           sortOrder
         }
       }
     }
   }
   ```

2. **Create a collection:**
   ```graphql
   mutation {
     collectionCreate(input: {
       title: "{title}",
       descriptionHtml: "{description}",
       ruleSet: {
         appliedDisjunctively: false,
         rules: [{column: TAG, relation: EQUALS, condition: "{tag}"}]
       }
     }) {
       collection { id title handle }
       userErrors { field message }
     }
   }
   ```

3. **Update collection sort order, rules, or products.**

4. **Publish/unpublish** collections to sales channels.

## Required Environment Variables

- `SHOPIFY_STORE_URL` — Shopify store URL
- `SHOPIFY_ACCESS_TOKEN` — Admin API access token

## Example Usage

```
User: /list-products tagged "winter-sale"
```

```
User: /check-orders from last 7 days
```

```
User: /check-inventory for low stock items
```

```
User: /update-prices apply 25% discount to all products tagged "seasonal"
```

```
User: /manage-collections create a "Spring Arrivals" collection
```
