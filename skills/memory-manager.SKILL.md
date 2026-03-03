---
name: memory-manager
description: Manage Claude Code's persistent memory in CLAUDE.md and topic files — save, search, audit, and export learned knowledge.
---

# Memory Manager

## Purpose

Manage Claude Code's persistent memory system. Claude Code retains knowledge across sessions by writing to `CLAUDE.md` and topic-specific files in the `.claude/` directory. This skill provides structured commands to save new learnings, search existing memory, audit for stale or conflicting entries, and export the knowledge base.

## When to Use

- The user says "save this to memory", "remember this", "search memory", or "what do you know about X".
- The user runs `/memory save`, `/memory search`, `/memory audit`, or `/memory export`.
- Another skill discovers something important (API change, error resolution, best practice) that should be persisted.
- The user wants to review what Claude Code has learned over time.

## Prerequisites

- **Playwright MCP server** must be installed and running. This is used when verifying that remembered API endpoints or versions are still current.
- A `CLAUDE.md` file in the project root.
- A `.claude/` directory for topic-specific memory files.

## Self-Learning Protocol

The memory manager is a core part of the self-learning system. It interacts with docs when:

1. **Auditing memory for staleness.** When running `/memory audit`, use Playwright to verify that API versions, endpoint URLs, and scope names stored in memory are still current:
   - Meta API: `https://developers.facebook.com/docs/graph-api/changelog`
   - Shopify API: `https://shopify.dev/docs/api/release-notes`

2. **Cross-referencing.** When saving new memory, check if it contradicts existing memory entries and resolve conflicts.

## Instructions

### `/memory save` — Save to Memory

1. **Accept the input.** The user provides a fact, lesson, or configuration to remember.

2. **Classify the memory** into a category:
   - `account-config` — IDs, credentials info (not secrets), account details
   - `api-lessons` — API quirks, error resolutions, version notes
   - `workflow-patterns` — successful multi-step processes
   - `platform-changes` — breaking changes, deprecations
   - `best-practices` — proven approaches for this project

3. **Check for duplicates.** Search CLAUDE.md and `.claude/` files for similar entries. If found, update rather than duplicate.

4. **Write the entry** to the appropriate location:
   - Core facts go in `CLAUDE.md` under the relevant section.
   - Topic-specific details go in `.claude/{topic}.md`.

5. **Format consistently:**
   ```markdown
   ### {Short Title}
   - **Saved**: {date}
   - **Category**: {category}
   - **Detail**: {the actual information}
   - **Source**: {how this was learned — user input, API error, docs, etc.}
   ```

### `/memory search` — Search Memory

1. **Accept a search query** from the user.
2. **Search across** `CLAUDE.md` and all `.claude/*.md` files.
3. **Return matching entries** with their location and context.
4. **If nothing found**, suggest related terms or offer to check external docs.

### `/memory audit` — Audit Memory

1. **Read all memory files** (`CLAUDE.md` + `.claude/*.md`).
2. **Check for:**
   - Duplicate entries (same fact in multiple places)
   - Contradictions (conflicting information)
   - Stale data (API versions that have been deprecated, old error workarounds)
   - Missing critical info (environment variables referenced but not documented)
3. **Use Playwright** to verify API versions and endpoints are still current.
4. **Report findings** with recommendations to fix.

### `/memory export` — Export Memory

1. **Compile all memory** into a single structured document.
2. **Organize by category** (account config, API lessons, workflows, etc.).
3. **Output** as a formatted markdown file.

## Required Environment Variables

- None — this skill reads and writes local files only.

## Example Usage

```
User: /memory save Meta API v21.0 deprecated the WEBSITE_CONVERSIONS objective, use OUTCOME_SALES instead
```

The skill will save this to the api-lessons section of CLAUDE.md.

```
User: /memory search rate limit
```

The skill will search all memory files for entries related to rate limits.

```
User: /memory audit
```

The skill will scan all memory files, check for stale API references, and report findings.
