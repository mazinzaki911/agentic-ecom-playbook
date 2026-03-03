---
name: error-handler
description: Structured error handling framework — diagnose API failures, check docs via Playwright if needed, apply fixes, and log resolutions to memory.
---

# Error Handler

## Purpose

Provide a systematic framework for diagnosing and resolving API errors across all platforms (Shopify, Meta, CDN, etc.). When any API call fails, this skill kicks in to classify the error, look up the error code in official documentation using Playwright, attempt automated fixes, and log the resolution to CLAUDE.md memory so the same error is resolved instantly next time.

## When to Use

- Any API call returns an error response (4xx, 5xx, or platform-specific error codes).
- The user says "debug this error", "why did this fail", or "fix this error".
- Another skill encounters an unexpected response and needs diagnosis.
- This skill is invoked automatically by other skills when they hit errors.

## Prerequisites

- **Playwright MCP server** must be installed and running. This is essential for this skill — it navigates to platform error documentation to look up error codes and find current solutions.
- At least one platform configured in `.env` (Shopify or Meta).

## Self-Learning Protocol

This skill IS the self-learning protocol. When an error occurs:

1. **Navigate to the platform's error reference:**
   - Meta Error Codes: `https://developers.facebook.com/docs/marketing-api/error-reference/`
   - Meta Graph API Errors: `https://developers.facebook.com/docs/graph-api/guides/error-handling/`
   - Shopify API Errors: `https://shopify.dev/docs/api/usage/response-codes`
   - Shopify GraphQL Errors: `https://shopify.dev/docs/api/usage/graphql-errors`

2. **Look up the specific error code.** Navigate to the error code page, read the description, cause, and recommended action.

3. **Check CLAUDE.md memory first.** Before hitting the docs, check if this error has been seen and resolved before. If so, apply the cached resolution.

4. **Apply the fix and log it.** After resolving, save the error code, cause, and resolution to CLAUDE.md so it is never looked up again.

## Instructions

1. **Capture the error.** Extract from the failed response:
   - HTTP status code
   - Platform error code (e.g., Meta error `17`, Shopify `THROTTLED`)
   - Error message / description
   - Request URL and parameters
   - Timestamp

2. **Check memory.** Search CLAUDE.md for the error code:
   ```
   ## Known Errors
   ### META-17: Rate Limit
   Resolution: Wait 5 minutes, then retry with exponential backoff.
   ```
   If found, apply the cached resolution immediately.

3. **Classify the error:**

   | Category | Codes | Action |
   |----------|-------|--------|
   | **Auth** | 190, 401, 403 | Re-authenticate, check token expiry |
   | **Rate Limit** | 17, 429, THROTTLED | Wait + exponential backoff |
   | **Invalid Params** | 100, 400, 422 | Check param names/types against docs |
   | **Not Found** | 803, 404 | Verify object ID exists |
   | **Permission** | 10, 294, 403 | Check scopes/permissions |
   | **Server** | 500, 502, 503 | Retry after delay |
   | **Deprecated** | various | Check API version, update endpoint |

4. **Look up in documentation** (if not in memory). Use Playwright to navigate to the error reference page and extract the current recommended action.

5. **Apply automated fix based on category:**
   - **Auth errors**: Check token expiry, prompt for new token if expired.
   - **Rate limits**: Implement exponential backoff (1s, 2s, 4s, 8s, max 60s).
   - **Invalid params**: Compare request params against current API docs.
   - **Not found**: Verify the object ID, check if it was deleted.
   - **Permission**: List current scopes, identify which is missing.
   - **Server errors**: Retry up to 3 times with 5s delay.
   - **Deprecated**: Find the replacement endpoint in the changelog.

6. **Log the resolution to CLAUDE.md:**
   ```markdown
   ### {PLATFORM}-{CODE}: {Short Description}
   - **Seen**: {date}
   - **Cause**: {root cause}
   - **Resolution**: {what fixed it}
   - **Prevention**: {how to avoid it}
   ```

7. **Report to the user:**
   ```
   Error Diagnosis
   ===============
   Platform: Meta Marketing API
   Code: 17 (Rate Limit Exceeded)
   Cause: Too many API calls in the current window
   Action: Waited 60s with exponential backoff, then retried successfully
   Logged: Yes (CLAUDE.md updated)
   ```

## Required Environment Variables

- None specific — this skill reads whatever platform credentials are in `.env`.

## Example Usage

```
User: I got error code 190 when trying to create a campaign
```

The skill will diagnose it as an expired Meta token, check the token debugger, and walk the user through getting a new token.

```
(Automatic) Another skill hits HTTP 429 from Shopify
```

The error handler applies exponential backoff, retries the request, and logs the rate limit pattern.
