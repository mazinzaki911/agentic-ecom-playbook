---
name: size-availability-auditor
description: Audit variant stock levels across size ranges to identify lucky sizes, out-of-stock gaps, and restock opportunities.
---

# Size Availability Auditor

## Purpose

Analyze inventory distribution across sizes for product collections. Identify "lucky sizes" (sizes with limited availability across many products), out-of-stock sizes, overstocked sizes, and size-specific restock needs. This data drives lucky-size promotions, targeted email campaigns, and inventory planning.

## When to Use

- The user asks "which sizes are running low?" or "what are the lucky sizes?"
- The user says "audit sizes", "check stock by size", or "size availability report".
- Before launching a lucky-sizes promotion to identify qualifying products.
- For inventory planning or restock decisions.
- The user wants to find products with only 1-2 sizes remaining.

## Instructions

1. **Define the scope.** Accept:
   - A collection handle or ID
   - A product type or tag filter
   - "All products" for a full catalog audit
   - A minimum/maximum stock threshold (default: flag sizes with <= 3 units)

2. **Fetch product and variant data:**
   ```graphql
   query ($cursor: String) {
     products(first: 50, after: $cursor, query: "{filter}") {
       pageInfo { hasNextPage endCursor }
       edges {
         node {
           id
           title
           handle
           productType
           tags
           totalInventory
           variants(first: 100) {
             edges {
               node {
                 id
                 title
                 sku
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

3. **Extract size data.** Parse the `selectedOptions` to find the size option:
   ```js
   const sizeOption = variant.selectedOptions.find(
     o => o.name.toLowerCase() === 'size' || o.name.toLowerCase() === 'مقاس'
   );
   const size = sizeOption?.value;
   ```
   Handle Arabic option names ("مقاس" = size).

4. **Build the size matrix.** Create a data structure:
   ```json
   {
     "sizeDistribution": {
       "39": { "totalStock": 45, "productsWithStock": 12, "productsOutOfStock": 3, "avgStock": 3.0 },
       "40": { "totalStock": 62, "productsWithStock": 14, "productsOutOfStock": 1, "avgStock": 4.1 },
       "41": { "totalStock": 78, "productsWithStock": 15, "productsOutOfStock": 0, "avgStock": 5.2 },
       "42": { "totalStock": 85, "productsWithStock": 15, "productsOutOfStock": 0, "avgStock": 5.7 },
       "43": { "totalStock": 55, "productsWithStock": 13, "productsOutOfStock": 2, "avgStock": 3.7 },
       "44": { "totalStock": 30, "productsWithStock": 8, "productsOutOfStock": 7, "avgStock": 2.0 },
       "45": { "totalStock": 12, "productsWithStock": 4, "productsOutOfStock": 11, "avgStock": 0.8 }
     }
   }
   ```

5. **Identify lucky-size products.** A "lucky size" product has:
   - Only 1-3 sizes remaining in stock (out of its normal size range)
   - At least 1 unit in stock for the remaining sizes
   ```js
   const luckyProducts = products.filter(p => {
     const inStockVariants = p.variants.filter(v => v.inventoryQuantity > 0);
     const totalVariants = p.variants.length;
     return inStockVariants.length >= 1 && inStockVariants.length <= 3 && totalVariants >= 5;
   });
   ```

6. **Generate the audit report:**
   ```
   Size Availability Audit Report
   ==============================
   Collection: Men's Leather Boots
   Products analyzed: 47
   Total variants: 282

   Size Distribution:
   Size | In Stock | OOS | Avg Qty | Status
   39   | 12/15    | 3   | 3.0     | LOW
   40   | 14/15    | 1   | 4.1     | OK
   41   | 15/15    | 0   | 5.2     | GOOD
   42   | 15/15    | 0   | 5.7     | GOOD
   43   | 13/15    | 2   | 3.7     | OK
   44   | 8/15     | 7   | 2.0     | LOW
   45   | 4/15     | 11  | 0.8     | CRITICAL

   Lucky Size Products (1-3 sizes left): 8 products
   - Classic Chelsea Boot: sizes 39, 44 remaining
   - Oxford Brogue: size 45 only
   ...

   Out of Stock Alert: 24 variants across 15 products
   Overstock Alert (>10 units): 12 variants across 8 products
   ```

7. **Export data.** Save results to:
   - `data/active/lucky-sizes-data.json` — structured data for downstream use
   - Optionally generate a CSV for spreadsheet analysis

## Required Environment Variables

- `SHOPIFY_STORE_URL` — Shopify store URL
- `SHOPIFY_ACCESS_TOKEN` — Admin API access token with `read_products`, `read_inventory` scopes

## Key Files

- `data/active/lucky-sizes-data.json` — lucky sizes analysis output
- `data/active/lucky-sizes-data.min.json` — minified version for theme use

## Example Usage

```
User: Audit size availability for the winter boots collection and find lucky-size products
```

The skill will fetch all variants, build the size matrix, identify lucky-size products, and present the report.
