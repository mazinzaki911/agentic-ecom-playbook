---
name: setup-project
description: Create the initial project structure (.env, package.json, .claude/ directory, CLAUDE.md) and guide the user through entering API credentials.
---

# Setup Project

## Purpose

Scaffold a new agentic e-commerce project from scratch. This skill creates all the files and directories Claude Code needs to operate as a full-stack e-commerce agent: environment configuration, package manifest, persistent memory directory, and the CLAUDE.md instructions file. It then walks the user through entering their API credentials so every other skill in the system can function.

## When to Use

- The user says "set up a new project", "initialize project", "start from scratch", or "scaffold".
- The current directory has no `.env`, `package.json`, or `CLAUDE.md`.
- A new team member needs to bootstrap their local environment.
- The user runs `/setup-project`.

## Prerequisites

- **Playwright MCP server** must be installed and running. This is what makes this skill future-proof — it can navigate to live platform documentation to verify current API setup steps, required scopes, and token generation flows before executing.
- Node.js >= 18 installed locally.
- The user must have credentials ready for at least one platform (Shopify or Meta).

## Self-Learning Protocol

Before executing any setup step that fails or seems outdated, use Playwright to verify:

1. **Navigate to the official docs:**
   - Shopify App Setup: `https://shopify.dev/docs/apps/getting-started`
   - Meta Developer Portal: `https://developers.facebook.com/docs/marketing-api/overview`
   - Meta Access Token Guide: `https://developers.facebook.com/docs/marketing-api/overview/authentication`

2. **Check the current requirements.** Both platforms change their onboarding flows. If a scope name changes or a new permission is required, read the current docs.

3. **Adapt the setup flow.** Update the required scopes, token generation steps, or environment variable names based on what the docs currently say.

4. **Save to memory.** Log any changes discovered to CLAUDE.md so future setups start with the correct info.

## Instructions

1. **Create the directory structure:**
   ```
   mkdir -p .claude data/active data/feeds scripts output
   ```

2. **Create `.env.example`** with all supported environment variables:
   ```env
   # Shopify
   SHOPIFY_STORE_URL=https://{your_store}.myshopify.com
   SHOPIFY_ACCESS_TOKEN=shpat_xxxxxxxxxxxxxxxxxxxxxxxxxx
   SHOPIFY_SHOP={your_store}.myshopify.com

   # Meta Marketing API
   META_ACCESS_TOKEN=EAAxxxxxxxxxx
   META_AD_ACCOUNT_ID=act_{your_ad_account_id}
   META_BUSINESS_ID={your_business_id}
   META_PIXEL_ID={your_pixel_id}
   META_CATALOG_ID={your_catalog_id}
   META_PAGE_ID={your_page_id}
   META_INSTAGRAM_ID={your_instagram_id}
   META_API_VERSION=v21.0

   # Store Config
   STORE_CURRENCY={your_currency}
   STORE_COUNTRY_CODE={your_country_code}
   STORE_TIMEZONE={your_timezone}
   ```

3. **Create `.env`** by copying `.env.example` and prompt the user for each value interactively.

4. **Create `.gitignore`:**
   ```
   .env
   node_modules/
   output/
   *.db
   *.db-shm
   *.db-wal
   .DS_Store
   ```

5. **Create `package.json`:**
   ```json
   {
     "name": "agentic-ecom-ops",
     "version": "1.0.0",
     "description": "Agentic e-commerce operations powered by Claude Code",
     "scripts": {
       "test": "echo \"Error: no test specified\" && exit 1"
     },
     "dependencies": {
       "dotenv": "^16.0.0",
       "node-fetch": "^2.7.0",
       "sharp": "^0.33.0",
       "p-limit": "^3.1.0"
     }
   }
   ```

6. **Create `CLAUDE.md`** with project overview, account configuration placeholders, and API reference sections.

7. **Create `.claude/` memory directory** with an initial `settings.json`.

8. **Run `npm install`** to install dependencies.

9. **Verify setup.** Check that:
   - `.env` exists and has non-empty values for at least one platform.
   - `node_modules/` was created successfully.
   - `CLAUDE.md` exists.

10. **Report results.** Print a summary of what was created and which credentials are still missing.

## Required Environment Variables

This skill *creates* environment variables rather than consuming them. The user will be prompted for:

- `SHOPIFY_STORE_URL`
- `SHOPIFY_ACCESS_TOKEN`
- `META_ACCESS_TOKEN`
- `META_AD_ACCOUNT_ID`
- `META_PIXEL_ID`
- `META_CATALOG_ID`
- `META_PAGE_ID`

## Example Usage

```
User: Set up a new project for my Shopify store
```

The skill will create all directories, config files, and walk the user through entering their Shopify credentials.

```
User: /setup-project
```

Full interactive scaffold with prompts for all platforms.
