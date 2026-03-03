---
name: cdn-image-uploader
description: Upload product images to Shopify CDN via the Files API (stagedUploadsCreate + file upload), returning permanent CDN URLs.
---

# CDN Image Uploader

## Purpose

Upload local image files to Shopify's CDN using the Admin GraphQL API's staged uploads mechanism. This produces permanent, publicly-accessible CDN URLs that can be used in Meta catalog feeds, ad creatives, and theme assets. Handles batch uploads with retry logic.

## When to Use

- The user has local images that need to be hosted on a CDN.
- The user says "upload images to CDN", "upload to Shopify", or "get CDN URLs".
- Generated offer/gallery images need permanent hosting before feed sync.
- Product images need to be uploaded for use in Meta ads.

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

1. **Identify images to upload.** Accept:
   - A directory path containing images (e.g., `output/gallery/`)
   - A list of specific file paths
   - A product-image-map file that references local files

2. **Create staged upload targets.** For each image, request a staged upload URL from Shopify:
   ```graphql
   mutation {
     stagedUploadsCreate(input: [
       {
         filename: "{filename}",
         mimeType: "image/jpeg",
         resource: FILE,
         fileSize: "{size_in_bytes}"
       }
     ]) {
       stagedTargets {
         url
         resourceUrl
         parameters {
           name
           value
         }
       }
       userErrors { field message }
     }
   }
   ```
   Batch up to 50 staged uploads per mutation call.

3. **Upload the file to the staged URL.** Each staged target provides a URL and form parameters. Upload via multipart form POST:
   ```js
   const FormData = require('form-data');
   const form = new FormData();

   // Add all parameters from the staged target
   for (const param of stagedTarget.parameters) {
     form.append(param.name, param.value);
   }
   // Add the file last
   form.append('file', fs.createReadStream(filePath));

   await fetch(stagedTarget.url, {
     method: 'POST',
     body: form,
     headers: form.getHeaders()
   });
   ```

4. **Register the file in Shopify.** After upload, create the file record:
   ```graphql
   mutation {
     fileCreate(files: [
       {
         alt: "{product_name} offer image",
         contentType: IMAGE,
         originalSource: "{resourceUrl_from_staged_target}"
       }
     ]) {
       files {
         ... on MediaImage {
           id
           image { url }
         }
       }
       userErrors { field message }
     }
   }
   ```

5. **Poll for the CDN URL.** File processing is async. Poll until the URL is ready:
   ```graphql
   query {
     node(id: "{file_id}") {
       ... on MediaImage {
         image { url }
         fileStatus
       }
     }
   }
   ```
   Wait for `fileStatus === "READY"`. The `url` is the permanent CDN URL.

6. **Build the upload map.** Save the mapping of filename to CDN URL:
   ```json
   {
     "123456_offer.jpg": "https://cdn.shopify.com/s/files/1/xxxx/files/123456_offer.jpg",
     "789012_offer.jpg": "https://cdn.shopify.com/s/files/1/xxxx/files/789012_offer.jpg"
   }
   ```
   Save to `data/active/gallery-upload-map.json` (or custom path).

7. **Handle failures.** Implement retry logic:
   - Retry failed uploads up to 3 times with exponential backoff
   - Track failed files separately
   - Report failed uploads at the end

8. **Rate limiting.** Respect Shopify's GraphQL rate limits:
   - Check `extensions.cost.throttleStatus.currentlyAvailable` in responses
   - If available points < 100, wait before next request
   - Process uploads sequentially or with limited concurrency (max 5 parallel)

## Required Environment Variables

- `SHOPIFY_STORE_URL` — Shopify store URL (e.g., `achilles-stores.myshopify.com`)
- `SHOPIFY_ACCESS_TOKEN` — Admin API access token with `write_files` scope

## Key Files

- `data/active/gallery-upload-map.json` — output mapping of filenames to CDN URLs
- `data/active/product-image-map.json` — input mapping of product IDs to image filenames

## Example Usage

```
User: Upload all images in output/gallery/ to Shopify CDN and save the URLs
```

The skill will create staged uploads, upload each file, poll for CDN URLs, and save the upload map.
