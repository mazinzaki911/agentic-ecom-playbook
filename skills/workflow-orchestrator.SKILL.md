---
name: workflow-orchestrator
description: Orchestrate multi-step agentic workflows that chain multiple skills together autonomously with pre-built workflows for common operations.
---

# Workflow Orchestrator

## Purpose

Orchestrate complex, multi-step workflows that chain multiple skills together in a defined sequence. Unlike the job-runner (which executes predefined sequences), the workflow orchestrator manages stateful, branching workflows where each step's output determines the next action. It includes pre-built workflows for common e-commerce operations and supports custom workflow definitions.

## When to Use

- The user says "run workflow", "launch product workflow", "daily ops workflow", or "orchestrate".
- The user runs `/workflow launch-product`, `/workflow daily-ops`, or `/workflow low-stock-react`.
- A complex multi-step operation needs autonomous execution with decision-making between steps.
- The user wants to define a custom workflow that chains skills with conditional logic.

## Prerequisites

- **Playwright MCP server** must be installed and running. This is used by individual skills in the workflow chain for self-learning.
- All skills referenced in the workflow must be available.
- Shopify Admin API access token in `.env`
- Meta Marketing API access token in `.env`

## Self-Learning Protocol

The orchestrator delegates to individual skills for API interactions. It uses Playwright when:

1. **Verifying workflow prerequisites.** Before starting a workflow, check that all required APIs and services are available:
   - Shopify API: `https://shopify.dev/docs/api/admin-graphql`
   - Meta API: `https://developers.facebook.com/docs/marketing-api/reference/`

2. **Handling workflow-level errors.** If a step fails and the error handler cannot resolve it, the orchestrator may need to look up alternative approaches in the documentation.

## Instructions

### `/workflow launch-product` — Product Launch Workflow

A complete workflow for launching new products from Shopify through to Meta advertising.

**Workflow Steps:**

```
Step 1: Product Discovery
├── Invoke: shopify-jobs-sop-pack (/list-products)
├── Input: product filter (tag, vendor, date)
├── Output: product list with variant IDs
└── Decision: if 0 products found → STOP

Step 2: Variant Mapping
├── Invoke: variant-mapper skill
├── Input: product list from Step 1
├── Output: variant-map.json
└── Decision: proceed

Step 3: Image Generation
├── Invoke: catalog-image-gen skill
├── Input: product list, overlay config
├── Output: generated images in output/gallery/
└── Decision: if any images failed → retry once, then proceed with successful ones

Step 4: CDN Upload
├── Invoke: cdn-image-uploader skill
├── Input: generated images
├── Output: CDN URLs in upload-map.json
└── Decision: if upload fails → retry with exponential backoff

Step 5: Feed Generation
├── Invoke: meta-feed-sync skill
├── Input: variant map + CDN URLs
├── Output: supplementary feed CSV
└── Decision: proceed

Step 6: Feed Upload to Meta
├── Invoke: meta-feed-sync skill (upload step)
├── Input: feed CSV
├── Output: upload confirmation
└── Decision: if upload fails → check error-handler, retry

Step 7: Product Set Creation
├── Invoke: collection-product-set-sync skill
├── Input: variant IDs for new products
├── Output: Meta product set ID
└── Decision: proceed

Step 8: Campaign Creation (optional)
├── Invoke: media-buying-sop-pack (/create-campaign)
├── Input: product set ID, budget, targeting
├── Output: campaign, ad set, ad IDs
└── Decision: only if user opted in
```

**Completion Report:**
```
Product Launch Workflow Complete
================================
Products: 12 launched
Images: 12 generated, 12 uploaded
Feed: 36 variant entries added
Product Set: Created (ID: {id})
Campaign: Created (PAUSED) — {campaign_name}

Duration: 4m 32s
Errors: 1 (image retry on product #7, resolved)
```

### `/workflow daily-ops` — Daily Operations Workflow

```
Step 1: Health Check → verify connections
Step 2: Order Check → new orders summary
Step 3: Inventory Scan → flag low/out-of-stock
Step 4: Ad Performance → pull today's metrics
Step 5: Low Stock React → if critical items OOS, pause related ads
Step 6: Generate Report → compile daily summary
```

### `/workflow low-stock-react` — Low Stock Reactive Workflow

Triggered when inventory drops below threshold:

```
Step 1: Identify affected products (inventory check)
Step 2: Find related Meta ads (search by product set)
Step 3: Pause ads for out-of-stock products
Step 4: Update catalog feed (mark as out of stock)
Step 5: Notify (generate alert report)
Step 6: Set monitoring (check again in 24h)
```

### `/workflow custom` — Custom Workflow

1. **Accept workflow definition** from the user:
   ```
   Steps:
   1. [skill-name] with [params]
   2. [skill-name] with [params] — depends on step 1 output
   3. IF [condition] THEN [skill-name] ELSE [skill-name]
   ```

2. **Parse and validate** the workflow definition.
3. **Execute** with state management between steps.
4. **Report** step-by-step results.

## Required Environment Variables

- `SHOPIFY_STORE_URL` — Shopify store URL
- `SHOPIFY_ACCESS_TOKEN` — Admin API access token
- `META_ACCESS_TOKEN` — Meta access token
- `META_AD_ACCOUNT_ID` — ad account ID
- `META_CATALOG_ID` — catalog ID

## Example Usage

```
User: /workflow launch-product for products tagged "spring-2026"
```

Runs the full product launch workflow from discovery through campaign creation.

```
User: /workflow daily-ops
```

Runs the daily operations workflow and generates a summary.

```
User: /workflow low-stock-react threshold=3
```

Scans inventory and reacts to any products below 3 units.
