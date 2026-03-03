---
name: theme-asset-deployer
description: Deploy Liquid snippets, section files, and template changes to Shopify themes via the Asset API, with backup and diff preview.
---

# Theme Asset Deployer

## Purpose

Push code changes (Liquid templates, CSS, JavaScript, section files, snippets) to a Shopify theme without using the Shopify CLI. This skill handles the Asset API workflow: listing existing assets, creating backups, uploading new or modified files, and verifying deployment. Useful for deploying sale banners, badge overlays, promotional sections, and landing pages.

## When to Use

- The user wants to push theme changes to Shopify.
- The user says "deploy theme files", "push badge update", "update the theme", or "deploy section".
- A Liquid snippet needs to be added or modified (e.g., sale badge, promo banner).
- Section files need to be deployed for a new landing page.
- The user wants to check what's currently on the theme before deploying.

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

1. **Identify the target theme.** List available themes and find the active one:
   ```
   GET /admin/api/2024-01/themes.json
   ```
   Look for the theme with `role: "main"` (the published theme). The user may also specify an unpublished theme for staging.

2. **List existing assets** to understand current state:
   ```
   GET /admin/api/2024-01/themes/{theme_id}/assets.json
   ```
   This returns all file paths in the theme. Filter to relevant directories:
   - `sections/` — section files
   - `snippets/` — reusable snippet files
   - `templates/` — page and product templates
   - `assets/` — CSS, JS, images
   - `layout/` — theme layout files

3. **Backup the existing file** before overwriting:
   ```
   GET /admin/api/2024-01/themes/{theme_id}/assets.json?asset[key]={file_path}
   ```
   Save the response body to a local backup file with timestamp:
   `backups/{file_path}_{timestamp}.liquid`

4. **Prepare the new file content.** Read the local Liquid/CSS/JS file. If the user provides inline code, write it to a temp file first. Validate Liquid syntax if possible (check for unclosed tags, missing `endfor`/`endif`).

5. **Deploy the asset.**
   ```
   PUT /admin/api/2024-01/themes/{theme_id}/assets.json
   {
     "asset": {
       "key": "snippets/sale-badge.liquid",
       "value": "{% if product.compare_at_price > product.price %}..."
     }
   }
   ```
   For binary files (images), use base64:
   ```json
   {
     "asset": {
       "key": "assets/promo-banner.png",
       "attachment": "{base64_encoded_content}"
     }
   }
   ```

6. **Deploy multiple files** in sequence. If deploying a section that references a snippet, deploy the snippet first:
   ```
   1. snippets/sale-badge.liquid
   2. sections/sale-banner.liquid
   3. templates/page.sale.json (if it references the section)
   ```

7. **Verify deployment.** Re-fetch the asset to confirm it was uploaded:
   ```
   GET /admin/api/2024-01/themes/{theme_id}/assets.json?asset[key]={file_path}
   ```
   Compare the returned content with what was deployed.

8. **Report results.** For each file:
   - Deployed path
   - Backup location
   - Status (created/updated)
   - File size

## Required Environment Variables

- `SHOPIFY_STORE_URL` — Shopify store URL
- `SHOPIFY_ACCESS_TOKEN` — Admin API access token with `write_themes` scope

## Safety Notes

- **Always backup** before overwriting. Theme changes are live immediately on the published theme.
- **Use an unpublished theme** for testing when possible.
- **Never deploy to layout/theme.liquid** without explicit user confirmation — breaking this file takes down the entire store.
- The REST API (not GraphQL) is used for theme assets.

## Example Usage

```
User: Deploy the updated sale-badge snippet and the winter-sale section to the live theme
```

The skill will backup existing files, deploy the snippet first, then the section, and verify both are live.
