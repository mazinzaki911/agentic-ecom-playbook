---
name: plugin-setup-wizard
description: Install and configure MCP servers (Playwright, Shopify Dev, etc.) in the project's .mcp.json and verify each plugin works.
---

# Plugin Setup Wizard

## Purpose

Install, configure, and verify all Model Context Protocol (MCP) server plugins that Claude Code needs to operate as a full-stack e-commerce agent. This includes the Playwright browser plugin (for self-learning), Shopify Dev MCP (for direct Shopify access), and any other MCP servers the user wants. The wizard writes the `.mcp.json` configuration and tests each plugin.

## When to Use

- The user says "set up plugins", "install MCP servers", "configure Playwright", or "set up MCP".
- Another skill's self-learning protocol fails because Playwright MCP is not installed.
- The user wants to add a new MCP server to the project.
- The user runs `/plugin-setup-wizard`.

## Prerequisites

- **Node.js >= 18** installed locally.
- **npx** available in PATH.
- Claude Code CLI installed and functional.

## Self-Learning Protocol

Before configuring any MCP server, use web search or documentation to verify:

1. **Navigate to MCP server registries and docs:**
   - MCP Protocol Spec: `https://modelcontextprotocol.io/`
   - Playwright MCP: `https://github.com/anthropics/mcp-playwright` (or current canonical source)
   - Shopify Dev MCP: search for the latest Shopify MCP server package

2. **Check the current installation method.** MCP server packages may change their npm package names, CLI flags, or configuration format.

3. **Read the configuration reference.** Verify the `.mcp.json` schema, required fields, and server launch commands.

4. **Adapt and update.** If package names or config formats have changed, update the wizard steps accordingly.

## Instructions

1. **Audit the current setup.** Check if `.mcp.json` already exists and what servers are configured:
   ```bash
   cat .mcp.json 2>/dev/null || echo "No .mcp.json found"
   ```

2. **Install Playwright MCP server** (required for self-learning):
   ```json
   {
     "mcpServers": {
       "playwright": {
         "command": "npx",
         "args": ["@anthropic/mcp-playwright"]
       }
     }
   }
   ```
   This is the most critical plugin — it enables every skill to navigate documentation and learn from the source.

3. **Install Shopify Dev MCP server** (if user has a Shopify store):
   ```json
   {
     "mcpServers": {
       "shopify-dev": {
         "command": "npx",
         "args": ["@anthropic/mcp-shopify-dev"],
         "env": {
           "SHOPIFY_STORE_URL": "${SHOPIFY_STORE_URL}",
           "SHOPIFY_ACCESS_TOKEN": "${SHOPIFY_ACCESS_TOKEN}"
         }
       }
     }
   }
   ```

4. **Install additional MCP servers** as requested (e.g., filesystem, database, custom servers).

5. **Write the `.mcp.json`** configuration file with all configured servers.

6. **Verify each plugin.** For each configured server:
   - Attempt to launch it and confirm it responds.
   - For Playwright: navigate to a test URL (e.g., `https://example.com`) and confirm a snapshot is returned.
   - For Shopify Dev: attempt a basic store query.

7. **Report results:**
   ```
   Plugin Setup Complete
   =====================
   Playwright MCP:   INSTALLED + VERIFIED
   Shopify Dev MCP:  INSTALLED + VERIFIED

   .mcp.json written with 2 servers configured.
   ```

## Required Environment Variables

- `SHOPIFY_STORE_URL` — (optional) needed for Shopify Dev MCP
- `SHOPIFY_ACCESS_TOKEN` — (optional) needed for Shopify Dev MCP

## Example Usage

```
User: Set up the Playwright and Shopify MCP plugins
```

The skill will install both MCP servers, write `.mcp.json`, and verify each one works.

```
User: /plugin-setup-wizard
```

Interactive wizard that asks which MCP servers to install.
