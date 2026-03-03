---
name: webhook-manager
description: Register, list, update, and delete Shopify webhooks via the Admin API for real-time event handling (orders, inventory, products).
---

# Webhook Manager

## Purpose

Manage Shopify webhooks that notify external services when events occur in the store. This includes registering new webhooks for order creation, inventory updates, product changes, and other events; listing active webhooks; and cleaning up stale registrations.

## When to Use

- The user wants to register a new webhook (e.g., "notify me when an order is placed").
- The user says "create webhook", "set up webhook", "list webhooks", or "delete webhook".
- An integration needs real-time notifications from Shopify.
- The user wants to audit which webhooks are currently active.
- Stale or broken webhook URLs need to be cleaned up.

## Instructions

1. **List existing webhooks** to see what's already registered:
   ```graphql
   query {
     webhookSubscriptions(first: 50) {
       edges {
         node {
           id
           topic
           endpoint {
             ... on WebhookHttpEndpoint {
               callbackUrl
             }
           }
           format
           createdAt
         }
       }
     }
   }
   ```
   Or via REST:
   ```
   GET /admin/api/2024-01/webhooks.json
   ```

2. **Register a new webhook.** Use GraphQL for webhook creation:
   ```graphql
   mutation {
     webhookSubscriptionCreate(
       topic: ORDERS_CREATE
       webhookSubscription: {
         callbackUrl: "https://your-endpoint.com/webhooks/orders"
         format: JSON
       }
     ) {
       webhookSubscription {
         id
         topic
         endpoint {
           ... on WebhookHttpEndpoint {
             callbackUrl
           }
         }
       }
       userErrors { field message }
     }
   }
   ```

   Common webhook topics:
   | Topic | Event |
   |-------|-------|
   | `ORDERS_CREATE` | New order placed |
   | `ORDERS_UPDATED` | Order modified |
   | `ORDERS_PAID` | Order payment confirmed |
   | `ORDERS_FULFILLED` | Order shipped |
   | `PRODUCTS_CREATE` | New product created |
   | `PRODUCTS_UPDATE` | Product modified |
   | `PRODUCTS_DELETE` | Product deleted |
   | `INVENTORY_LEVELS_UPDATE` | Stock level changed |
   | `COLLECTIONS_UPDATE` | Collection modified |
   | `CUSTOMERS_CREATE` | New customer registered |
   | `CHECKOUTS_UPDATE` | Checkout modified |
   | `APP_UNINSTALLED` | App removed from store |

3. **Test the webhook endpoint** before registering. Send a test payload to the callback URL:
   ```js
   const testPayload = {
     id: 0,
     email: "test@example.com",
     note: "Webhook registration test"
   };
   const response = await fetch(callbackUrl, {
     method: 'POST',
     headers: { 'Content-Type': 'application/json' },
     body: JSON.stringify(testPayload)
   });
   ```
   Verify the endpoint returns 200. Shopify will disable webhooks that consistently fail.

4. **Update an existing webhook** (change URL or format):
   ```graphql
   mutation {
     webhookSubscriptionUpdate(
       id: "gid://shopify/WebhookSubscription/{id}"
       webhookSubscription: {
         callbackUrl: "https://new-endpoint.com/webhooks/orders"
       }
     ) {
       webhookSubscription { id }
       userErrors { field message }
     }
   }
   ```

5. **Delete a webhook:**
   ```graphql
   mutation {
     webhookSubscriptionDelete(id: "gid://shopify/WebhookSubscription/{id}") {
       deletedWebhookSubscriptionId
       userErrors { field message }
     }
   }
   ```

6. **Webhook verification.** When receiving webhooks in production, verify the HMAC signature:
   ```js
   const crypto = require('crypto');
   function verifyWebhook(body, hmacHeader, secret) {
     const hash = crypto.createHmac('sha256', secret)
       .update(body, 'utf8')
       .digest('base64');
     return hash === hmacHeader;
   }
   // Check: req.headers['x-shopify-hmac-sha256']
   ```

7. **Audit and clean up.** List all webhooks and identify:
   - Duplicate registrations (same topic + URL)
   - Webhooks pointing to dead URLs
   - Webhooks for uninstalled apps
   Report findings and offer to delete stale ones.

8. **Report results.** Show a table of all active webhooks:
   ```
   Active Webhooks
   ===============
   Topic                  | URL                                    | Format | Created
   ORDERS_CREATE          | https://api.example.com/webhooks/order | JSON   | 2026-01-15
   INVENTORY_LEVELS_UPDATE| https://api.example.com/webhooks/stock | JSON   | 2026-02-01
   PRODUCTS_UPDATE        | https://api.example.com/webhooks/prod  | JSON   | 2026-02-20
   ```

## Required Environment Variables

- `SHOPIFY_STORE_URL` — Shopify store URL
- `SHOPIFY_ACCESS_TOKEN` — Admin API access token with `read_webhooks`, `write_webhooks` scopes
- `SHOPIFY_WEBHOOK_SECRET` — (optional) for HMAC verification of incoming webhooks

## Example Usage

```
User: Register a webhook for order creation that posts to https://my-api.com/webhooks/new-order
```

The skill will check for existing registrations on that topic, create the webhook, and verify the endpoint is reachable.

```
User: List all active webhooks and clean up duplicates
```

The skill will fetch all webhooks, identify duplicates or stale entries, and offer to delete them.
