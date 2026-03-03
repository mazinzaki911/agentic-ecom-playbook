---
name: sop-to-skill-converter
description: Convert a written SOP document into a Claude Code SKILL.md file with proper frontmatter, instructions, and slash commands.
---

# SOP-to-Skill Converter

## Purpose

Transform any Standard Operating Procedure (SOP) document into a properly formatted Claude Code SKILL.md file. The converter reads an SOP (whether written by the sop-builder skill or manually), restructures it into the SKILL.md format with frontmatter, self-learning protocol, and slash command definitions, and outputs a ready-to-use skill file.

## When to Use

- The user says "convert this SOP to a skill", "make a skill from this SOP", or "turn this procedure into a skill file".
- The user runs `/sop-to-skill`.
- An SOP has been created by the sop-builder and the user wants to make it executable.
- The user has an existing markdown procedure document they want to standardize.

## Prerequisites

- **Playwright MCP server** must be installed and running. This is used to verify any API endpoints referenced in the SOP are current before encoding them into the skill.
- An existing SOP document (markdown file or inline text).

## Self-Learning Protocol

When converting an SOP that references API endpoints, use Playwright to verify:

1. **Navigate to the relevant platform docs:**
   - Meta Marketing API: `https://developers.facebook.com/docs/marketing-api/reference/`
   - Shopify Admin API: `https://shopify.dev/docs/api/admin-graphql`

2. **Verify all API calls in the SOP** are current — correct endpoints, parameters, and field names.

3. **Update any outdated references** before encoding them into the skill.

## Instructions

1. **Read the source SOP.** Accept input as:
   - A file path to an existing SOP markdown file.
   - Inline text pasted by the user.
   - A reference to an SOP created in the current session.

2. **Parse the SOP structure.** Extract:
   - **Title** — becomes the skill name
   - **Purpose/Description** — becomes the frontmatter description
   - **Prerequisites** — preserved and augmented with Playwright requirement
   - **Steps** — become the Instructions section
   - **Troubleshooting** — informs the Error Handling section
   - **Platform references** — determine the Self-Learning Protocol URLs

3. **Generate the SKILL.md frontmatter:**
   ```yaml
   ---
   name: {kebab-case-name}
   description: {one-line description from the SOP}
   ---
   ```

4. **Generate the Self-Learning Protocol.** Based on which platforms the SOP references:
   - **Shopify only**: Point to Shopify docs
   - **Meta only**: Point to Meta docs
   - **Both**: Point to both sets of docs
   - **Neither**: Generic protocol pointing to relevant external docs

5. **Convert steps to Instructions format:**
   - Number each step clearly.
   - Include code blocks for API calls.
   - Add "Expected Result" notes after each step.
   - Add error handling guidance inline.

6. **Generate slash command definitions.** If the SOP has a natural trigger command:
   ```markdown
   ## When to Use
   - The user runs `/{command-name}`
   - The user says "{natural language triggers}"
   ```

7. **Add required sections:**
   - Required Environment Variables
   - Example Usage
   - Key Files (if applicable)

8. **Validate the output.** Ensure:
   - Frontmatter is valid YAML.
   - All code blocks have language annotations.
   - No broken markdown formatting.
   - All API endpoints are verified current.

9. **Save the skill file** to the skills directory: `skills/{name}.SKILL.md`

10. **Report results:**
    ```
    SOP Converted to Skill
    ======================
    Source: docs/sops/upload-images.md
    Output: skills/upload-images.SKILL.md

    Name: upload-images
    Command: /upload-images
    Platform: Shopify + Meta
    Steps: 8
    Env Vars: 4 required

    The skill is ready to use.
    ```

## Required Environment Variables

- None — this skill reads and writes local files only.

## Example Usage

```
User: /sop-to-skill docs/sops/catalog-feed-update.md
```

The skill reads the SOP file, converts it to SKILL.md format, and saves the result.

```
User: Convert this SOP to a skill: [pastes SOP content]
```

The skill parses the inline content and generates a SKILL.md file.
