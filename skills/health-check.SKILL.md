---
name: health-check
description: Run a comprehensive health check — verify API connections (Shopify, Meta), MCP plugins, environment variables, and memory integrity.
---

# Health Check

## Purpose

Run a comprehensive diagnostic across the entire agentic e-commerce system. This skill verifies that all API connections are live, MCP plugins are functional, environment variables are correctly set, data files are present, and the CLAUDE.md memory is consistent. It produces a single status report that shows what is working and what needs attention.

## When to Use

- The user says "health check", "system status", "check everything", or "diagnose".
- At the start of a new session to verify all connections are live.
- After updating credentials, API versions, or plugin configurations.
- When something is failing and the user is unsure what is broken.
- The user runs `/health-check`.

## Prerequisites

- **Playwright MCP server** must be installed and running. This is used to verify that API documentation URLs are reachable and to check the current API version status.
- At least a partial `.env` file with some credentials configured.

## Self-Learning Protocol

During the health check, use Playwright to verify:

1. **API version status:**
   - Meta: `https://developers.facebook.com/docs/graph-api/changelog` — check if the configured version is still supported.
   - Shopify: `https://shopify.dev/docs/api/release-notes` — check if the configured version is still supported.

2. **Documentation accessibility.** Verify that the key documentation URLs used by the self-learning protocol are reachable.

## Instructions

1. **Check environment variables.** Read `.env` and verify:
   ```
   [ ] SHOPIFY_STORE_URL — present and non-empty
   [ ] SHOPIFY_ACCESS_TOKEN — present and non-empty
   [ ] META_ACCESS_TOKEN — present and non-empty
   [ ] META_AD_ACCOUNT_ID — present and starts with "act_"
   [ ] META_PIXEL_ID — present and non-empty
   [ ] META_CATALOG_ID — present and non-empty
   [ ] META_PAGE_ID — present and non-empty
   [ ] META_API_VERSION — present and matches v{number}.0 format
   ```

2. **Test Shopify connection:**
   ```graphql
   query { shop { name currencyCode } }
   ```
   Expected: 200 OK with shop data.

3. **Test Meta connection:**
   ```
   GET /me?fields=id,name&access_token={token}
   GET /act_{id}?fields=name,account_status,currency
   ```
   Expected: 200 OK with account data.

4. **Check Meta token expiry:**
   ```
   GET /debug_token?input_token={token}&access_token={token}
   ```
   Report days until expiration. Warn if < 7 days.

5. **Check MCP plugins.** Verify `.mcp.json` exists and each configured server is launchable.

6. **Check data files.** Verify key data files exist:
   ```
   [ ] data/active/products-to-process.json
   [ ] data/active/product-image-map.json
   [ ] data/active/gallery-upload-map.json
   [ ] data/feeds/meta-supplementary-feed-synced.csv
   ```

7. **Check memory integrity.** Verify:
   - `CLAUDE.md` exists and is non-empty.
   - `.claude/` directory exists.
   - No contradictory entries in memory (optional deep check).

8. **Generate the health report:**
   ```
   System Health Check
   ===================

   Connections
   -----------
   Shopify API:    [PASS] Connected to {store_name} ({currency})
   Meta API:       [PASS] Connected to {account_name}
   Meta Token:     [WARN] Expires in 12 days

   MCP Plugins
   -----------
   Playwright:     [PASS] Running
   Shopify Dev:    [FAIL] Not configured

   Environment
   -----------
   .env:           [PASS] 10/12 variables set
   Missing:        META_INSTAGRAM_ID, META_BUSINESS_ID

   Data Files
   ----------
   products-to-process.json:    [PASS] 47 products
   product-image-map.json:      [PASS] 47 entries
   gallery-upload-map.json:     [PASS] 141 URLs

   Memory
   ------
   CLAUDE.md:      [PASS] 245 lines
   .claude/:       [PASS] 3 topic files

   Overall: 8/10 checks passed (1 warning, 1 failure)
   ```

## Required Environment Variables

This skill reads all environment variables to check them. It does not require any specific ones — missing variables are reported as findings.

## Example Usage

```
User: Run a health check
```

The skill will test all connections, verify all config, and produce a comprehensive status report.

```
User: /health-check
```

Same as above — full system diagnostic.
