---
name: shopify-connect
description: Connect Claude Code to a Shopify store via the Admin API — test the connection, verify scopes, and store credentials in .env.
---

# Shopify Connect

## Purpose

Establish and verify a live connection between Claude Code and a Shopify store. This skill validates that the Admin API access token works, checks which API scopes are granted, confirms the store URL resolves, and persists the credentials in `.env`. It is the prerequisite gate for every Shopify skill in the system.

## When to Use

- The user says "connect to Shopify", "test Shopify connection", or "set up Shopify".
- Another skill fails with a Shopify authentication error and the user needs to re-establish the connection.
- The user has a new access token and needs to update credentials.
- The user runs `/shopify-connect`.

## Prerequisites

- **Playwright MCP server** must be installed and running. This is what makes this skill future-proof — it can navigate to Shopify's developer docs to verify current authentication requirements, scope names, and API versions.
- A Shopify store with a Dev Dashboard app (client credentials grant) installed on it.
- The store URL (e.g., `{your_store}.myshopify.com`).
- The Dev Dashboard app must be configured with:
  - **App URL**: `http://localhost:3000`
  - **Redirect URL**: `http://localhost:3000/callback`
  - **Embed app in Shopify admin**: unchecked

## Self-Learning Protocol

Before executing any API call that fails or seems outdated, use Playwright to verify:

1. **Navigate to the official docs:**
   - Shopify Admin API Authentication: `https://shopify.dev/docs/api/admin-graphql`
   - Shopify Access Scopes: `https://shopify.dev/docs/api/usage/access-scopes`
   - API Versioning: `https://shopify.dev/docs/api/usage/versioning`

2. **Check the current API version.** Shopify deprecates versions quarterly. Verify which versions are currently supported and which is the latest stable.

3. **Read the authentication reference.** Check if authentication headers, token formats, or scope names have changed.

4. **Adapt and retry.** Update the connection test with current parameters and re-execute. Save what you learned to CLAUDE.md memory.

## Instructions

1. **Collect credentials.** Ask the user for:
   - Store URL: `https://{your_store}.myshopify.com`
   - Client ID from the Dev Dashboard
   - Client Secret from the Dev Dashboard
   - Redirect URI (default: `http://localhost:3000/callback`)

2. **Test the connection** with a lightweight GraphQL query:
   ```graphql
   query {
     shop {
       name
       primaryDomain { url }
       plan { displayName }
       currencyCode
     }
   }
   ```
   Send via:
   ```bash
   curl -s -X POST "https://{your_store}.myshopify.com/admin/api/2024-10/graphql.json" \
     -H "X-Shopify-Access-Token: {token}" \
     -H "Content-Type: application/json" \
     -d '{"query": "{ shop { name primaryDomain { url } plan { displayName } currencyCode } }"}'
   ```

3. **Verify API scopes.** Check which scopes the token has:
   ```
   GET /admin/oauth/access_scopes.json
   ```
   Compare against the required scopes for the playbook:
   - `read_products`, `write_products`
   - `read_orders`
   - `read_inventory`, `write_inventory`
   - `read_files`, `write_files`
   - `read_content`, `write_content`
   - `read_themes`, `write_themes`

4. **Report missing scopes.** If critical scopes are missing, list them and explain how to add them in the Shopify admin under Apps > [App Name] > API Scopes.

5. **Save credentials to `.env`:**
   ```env
   SHOPIFY_SHOP={your_store}
   SHOPIFY_CLIENT_ID=your-client-id
   SHOPIFY_CLIENT_SECRET=your-client-secret
   SHOPIFY_REDIRECT_URI=http://localhost:3000/callback
   ```

6. **Update CLAUDE.md** with the store name, plan, and currency for future reference.

7. **Report results:**
   ```
   Shopify Connection: OK
   Store: {store_name}
   Plan: {plan_name}
   Currency: {currency_code}
   Scopes: 12/14 required scopes granted
   Missing: write_themes, read_content
   ```

## Required Environment Variables

After this skill runs, these will be set:

- `SHOPIFY_SHOP` — store handle (without `.myshopify.com`)
- `SHOPIFY_CLIENT_ID` — Client ID from Dev Dashboard
- `SHOPIFY_CLIENT_SECRET` — Client Secret from Dev Dashboard
- `SHOPIFY_REDIRECT_URI` — OAuth callback URL (`http://localhost:3000/callback`)

## Example Usage

```
User: Connect to my Shopify store at mystore.myshopify.com
```

The skill will prompt for Client ID and Secret, test the connection via client credentials grant, verify scopes, save credentials, and report the store details.

```
User: /shopify-connect
```

Interactive flow that prompts for the store URL and Dev Dashboard credentials.
