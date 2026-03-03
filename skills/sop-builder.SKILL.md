---
name: sop-builder
description: Interactive SOP creation wizard — asks 5 questions (task, platform, prerequisites, steps, errors) then generates a formatted SOP document.
---

# SOP Builder

## Purpose

Create standardized Standard Operating Procedure (SOP) documents through an interactive wizard. The wizard asks 5 structured questions to gather all the information needed, then generates a fully formatted SOP document that can be used by humans or converted into a Claude Code skill file.

## When to Use

- The user says "create an SOP", "build a procedure", "document this process", or "write an SOP for X".
- The user has a repeatable process they want to standardize.
- The user runs `/build-sop`.

## Prerequisites

- **Playwright MCP server** must be installed and running. This is used to verify API endpoints and platform-specific steps referenced in the SOP against current documentation.

## Self-Learning Protocol

When building an SOP that references API endpoints or platform features, use Playwright to verify:

1. **Navigate to the relevant platform docs:**
   - Meta Marketing API: `https://developers.facebook.com/docs/marketing-api/reference/`
   - Shopify Admin API: `https://shopify.dev/docs/api/admin-graphql`

2. **Verify that all API calls in the SOP steps are current** — correct endpoint URLs, parameter names, and required fields.

3. **Check for any recent changes** that might affect the procedure.

## Instructions

1. **Start the wizard.** Present the 5 questions to the user one at a time:

   **Question 1: What is the task?**
   > "What process or task does this SOP cover? Describe it in one sentence."
   > Example: "Upload supplementary product images to Meta catalog via API"

   **Question 2: Which platform(s)?**
   > "Which platforms are involved? (Shopify, Meta, Both, Other)"
   > This determines which API docs to reference.

   **Question 3: What are the prerequisites?**
   > "What must be true before starting? (credentials, data files, previous steps)"
   > Example: "Need META_ACCESS_TOKEN, product images already uploaded to CDN"

   **Question 4: What are the steps?**
   > "Walk me through each step, numbered. I will help format them."
   > Collect raw steps, then enrich with API call details.

   **Question 5: What can go wrong?**
   > "What errors or edge cases have you encountered? How were they resolved?"
   > This populates the troubleshooting section.

2. **Verify steps against docs.** For each step that involves an API call, use Playwright to navigate to the relevant documentation and confirm the endpoint, parameters, and expected response.

3. **Generate the SOP document** in this format:
   ```markdown
   # SOP: {Task Title}

   **Last Updated:** {date}
   **Platform:** {platform}
   **Estimated Time:** {estimate}

   ## Prerequisites
   - {prerequisite 1}
   - {prerequisite 2}

   ## Procedure

   ### Step 1: {Step Title}
   {Description}
   ```{code block if applicable}```
   **Expected Result:** {what success looks like}

   ### Step 2: {Step Title}
   ...

   ## Troubleshooting

   | Error | Cause | Resolution |
   |-------|-------|------------|
   | {error 1} | {cause} | {fix} |

   ## Notes
   - {any additional context}
   ```

4. **Save the SOP** to the appropriate location (e.g., `docs/sops/{sop-name}.md` or a location the user specifies).

5. **Offer conversion.** Ask if the user wants to convert this SOP into a SKILL.md file using the `sop-to-skill-converter` skill.

## Required Environment Variables

- None — this skill generates documents, it does not make API calls.

## Example Usage

```
User: /build-sop
```

The wizard starts and asks the 5 questions interactively.

```
User: Create an SOP for uploading product images to Meta catalog
```

The wizard pre-fills "upload product images to Meta catalog" as the task and proceeds to the remaining questions.
