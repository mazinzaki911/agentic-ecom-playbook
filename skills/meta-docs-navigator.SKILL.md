---
name: meta-docs-navigator
description: Use Playwright to navigate Meta's developer documentation, read API references, and extract current endpoint specs.
---

# Meta Docs Navigator

## Purpose

Provide a reliable way to look up current Meta Marketing API documentation using the Playwright browser. Instead of relying on potentially outdated training data, this skill navigates directly to Meta's developer docs, reads the live content, and extracts the current endpoint specifications, parameters, and examples. It is the foundation of the self-learning protocol for all Meta-related skills.

## When to Use

- The user says "check Meta docs", "look up the Meta API for X", or "what's the current spec for Y endpoint".
- Any Meta skill needs to verify an endpoint before making an API call.
- A Meta API call fails and the error handler needs to look up the error code.
- Meta has released a new API version and skills need to update.
- The user runs `/read-meta-docs`.

## Prerequisites

- **Playwright MCP server** must be installed and running. This skill is entirely dependent on Playwright — it IS the browser-based documentation reader.

## Self-Learning Protocol

This skill IS the self-learning protocol for Meta. It:

1. **Navigates to Meta's developer documentation:**
   - Marketing API Reference: `https://developers.facebook.com/docs/marketing-api/reference/`
   - Graph API Reference: `https://developers.facebook.com/docs/graph-api/reference/`
   - Error Reference: `https://developers.facebook.com/docs/marketing-api/error-reference/`
   - Changelog: `https://developers.facebook.com/docs/graph-api/changelog`
   - Deprecation Schedule: `https://developers.facebook.com/docs/graph-api/guides/versioning`

2. **Reads the page content** using Playwright's snapshot or evaluate capabilities.

3. **Extracts structured information:**
   - Endpoint URL pattern
   - Required and optional parameters
   - Request/response examples
   - Error codes specific to the endpoint
   - API version requirements

4. **Returns the information** in a structured format that other skills can consume.

## Instructions

1. **Accept the query.** The user or another skill specifies what they need to look up:
   - A specific endpoint (e.g., "campaign creation endpoint")
   - An error code (e.g., "error code 17")
   - A concept (e.g., "custom audiences")
   - The changelog for a specific version

2. **Determine the documentation URL.** Map the query to the right docs page:
   | Query Type | URL Pattern |
   |-----------|-------------|
   | Campaign endpoint | `https://developers.facebook.com/docs/marketing-api/reference/ad-campaign-group/` |
   | Ad Set endpoint | `https://developers.facebook.com/docs/marketing-api/reference/ad-campaign/` |
   | Ad endpoint | `https://developers.facebook.com/docs/marketing-api/reference/adgroup/` |
   | Creative endpoint | `https://developers.facebook.com/docs/marketing-api/reference/ad-creative/` |
   | Product Catalog | `https://developers.facebook.com/docs/marketing-api/reference/product-catalog/` |
   | Insights | `https://developers.facebook.com/docs/marketing-api/reference/ad-account/insights/` |
   | Error codes | `https://developers.facebook.com/docs/marketing-api/error-reference/` |
   | Changelog | `https://developers.facebook.com/docs/graph-api/changelog` |

3. **Navigate with Playwright:**
   ```
   browser_navigate -> {url}
   browser_snapshot -> capture the page content
   ```

4. **Extract relevant information** from the snapshot:
   - Look for parameter tables, code examples, and descriptions.
   - Parse the structured content into a usable format.

5. **Return the result** in a standardized format:
   ```
   Meta API Reference: {endpoint_name}
   =====================================
   URL: POST /act_{ad_account_id}/campaigns
   API Version: v21.0+

   Required Parameters:
   - name (string): Campaign name
   - objective (enum): OUTCOME_SALES, OUTCOME_LEADS, ...
   - status (enum): ACTIVE, PAUSED
   - special_ad_categories (array): []

   Optional Parameters:
   - daily_budget (int): Daily budget in minor currency units
   - lifetime_budget (int): Lifetime budget in minor currency units
   - bid_strategy (enum): LOWEST_COST_WITHOUT_CAP, ...

   Example:
   POST /act_123/campaigns
   {"name":"Test","objective":"OUTCOME_SALES","status":"PAUSED","special_ad_categories":[]}
   ```

6. **Cache the result** in CLAUDE.md memory for faster lookups in the same session.

## Required Environment Variables

- None — this skill only reads documentation, it does not make API calls.

## Example Usage

```
User: /read-meta-docs campaign creation endpoint
```

The skill will navigate to the campaign creation API reference, extract the current spec, and return it.

```
User: What parameters does the Meta ad creative endpoint accept?
```

The skill will look up the ad creative reference page and return the parameter list.
