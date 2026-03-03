---
name: meta-connect
description: Connect Claude Code to a Meta ad account via the Marketing API — test token validity, verify permissions, and store credentials in .env.
---

# Meta Connect

## Purpose

Establish and verify a live connection between Claude Code and a Meta (Facebook/Instagram) ad account. This skill validates the access token, checks granted permissions, discovers the ad account details (name, currency, timezone), and persists all credentials in `.env`. It is the prerequisite gate for every Meta advertising skill in the system.

## When to Use

- The user says "connect to Meta", "test Meta connection", "set up Meta ads", or "connect Facebook ads".
- Another skill fails with a Meta authentication error (`190` - token expired).
- The user has a new access token and needs to update credentials.
- The user runs `/meta-connect`.

## Prerequisites

- **Playwright MCP server** must be installed and running. This is what makes this skill future-proof — it can navigate to Meta's developer documentation to verify current authentication flows, token types, and permission requirements.
- A Meta Business account with a configured Ad Account.
- A User Access Token (long-lived preferred) with `ads_management` and `ads_read` permissions.

## Self-Learning Protocol

Before executing any API call that fails or seems outdated, use Playwright to verify:

1. **Navigate to the official docs:**
   - Meta Marketing API Overview: `https://developers.facebook.com/docs/marketing-api/overview`
   - Authentication Guide: `https://developers.facebook.com/docs/marketing-api/overview/authentication`
   - Access Token Debugger: `https://developers.facebook.com/tools/debug/accesstoken/`
   - Graph API Changelog: `https://developers.facebook.com/docs/graph-api/changelog`

2. **Check the current API version.** Meta deprecates API versions on a rolling basis. Verify which versions are active and which is recommended.

3. **Read the permissions reference.** Check if permission names or requirements have changed for the Marketing API.

4. **Adapt and retry.** Update the connection test with current parameters and re-execute. Save what you learned to CLAUDE.md memory.

## Instructions

1. **Collect credentials.** Ask the user for:
   - Meta Access Token (User or System User token)
   - Ad Account ID (format: `act_XXXXXXXXX`)

2. **Debug the token** to check validity and permissions:
   ```
   GET /debug_token?input_token={token}&access_token={token}
   ```
   Extract: `is_valid`, `expires_at`, `scopes`, `type`.

3. **Check token expiration.** If the token expires within 7 days, warn the user. If it is a short-lived token (~1 hour), offer to exchange it for a long-lived token:
   ```
   GET /oauth/access_token?grant_type=fb_exchange_token&client_id={app_id}&client_secret={app_secret}&fb_exchange_token={short_token}
   ```

4. **Verify ad account access:**
   ```
   GET /act_{ad_account_id}?fields=name,account_status,currency,timezone_name,business
   ```
   Confirm the token can access this specific ad account.

5. **Discover associated assets.** Fetch related IDs:
   ```
   GET /act_{ad_account_id}?fields=name,currency,timezone_name
   GET /act_{ad_account_id}/adspixels?fields=id,name
   GET /act_{ad_account_id}/product_catalogs?fields=id,name,product_count
   GET /{business_id}/pages?fields=id,name
   GET /{page_id}?fields=instagram_business_account
   ```

6. **Save credentials to `.env`:**
   ```env
   META_ACCESS_TOKEN=EAAxxxxxxxxxx
   META_AD_ACCOUNT_ID=act_{ad_account_id}
   META_BUSINESS_ID={business_id}
   META_PIXEL_ID={pixel_id}
   META_CATALOG_ID={catalog_id}
   META_PAGE_ID={page_id}
   META_INSTAGRAM_ID={instagram_id}
   META_API_VERSION=v21.0
   ```

7. **Update CLAUDE.md** with the ad account name, currency, timezone, and discovered asset IDs.

8. **Report results:**
   ```
   Meta Connection: OK
   Ad Account: {account_name} (act_{id})
   Status: ACTIVE
   Currency: {currency}
   Timezone: {timezone}
   Token Expires: {date} ({days} days remaining)
   Pixel: {pixel_name} ({pixel_id})
   Catalog: {catalog_name} ({product_count} products)
   Page: {page_name}
   Instagram: {instagram_id}
   ```

## Required Environment Variables

After this skill runs, these will be set:

- `META_ACCESS_TOKEN` — long-lived User Access Token
- `META_AD_ACCOUNT_ID` — ad account ID (with `act_` prefix)
- `META_BUSINESS_ID` — business manager ID
- `META_PIXEL_ID` — pixel ID
- `META_CATALOG_ID` — product catalog ID
- `META_PAGE_ID` — Facebook Page ID
- `META_INSTAGRAM_ID` — Instagram account ID
- `META_API_VERSION` — API version string

## Example Usage

```
User: Connect to my Meta ad account act_123456789 with this token: EAAxxxxxx
```

The skill will validate the token, discover all associated assets, save credentials, and report the full account details.

```
User: /meta-connect
```

Interactive flow that prompts for token and ad account ID.
