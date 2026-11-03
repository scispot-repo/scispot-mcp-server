# Scispot MCP Server

Connect Claude, Cursor, or another MCP client to your Scispot lab workspace — query Labsheets, manage ELN experiments, protocols, documentation, storage locations, manifests, and more, all through natural language.

## Use with Cursor

This repository is also a [Cursor plugin](https://cursor.com/docs/reference/plugins). It connects Cursor's agent to Scispot's hosted MCP server at `https://mcp.scispot.io/` (streamable HTTP).

### Install

1. Open **Cursor Settings → Plugins**.
2. Search for **Scispot** and click **Install**.
3. When Cursor connects to the server, it opens the Scispot sign-in page in your browser. Sign in with your Scispot account to authorize the connection (OAuth). There is no API key to paste into Cursor.

Or run `/add-plugin scispot` in chat.

The plugin adds this MCP server:

```json
{
  "mcpServers": {
    "scispot": {
      "type": "http",
      "url": "https://mcp.scispot.io/"
    }
  }
}
```

### Requirements

- A Scispot account with Super Admin access, or an account with Dry Lab access.
- Tool calls run with your Scispot permissions. The agent can see and change only what your account allows.

### Example prompts

- "List my Scispot Labsheets."
- "Show the schema of the 'Cell Lines' Labsheet, then find rows where Status is 'Active'."
- "List the protocols in the 'Genomics' Labspace and summarize the 'DNA Extraction SOP' protocol."
- "Look up the manifest with barcode 'PLT-00142' and show which wells are filled."
- "Create an experiment called 'Western Blot Optimization Round 3' in the 'Protein Lab' Labspace and add a summary of today's run."

Start with a read-only request such as "List my Scispot Labsheets" to confirm the connection.

### Write tools

Some tools change data in your workspace: `add_labsheet_rows`, `update_labsheet_rows`, `create_*_stub`, `write_to_*`, and `link_labsheet_to_*`. `update_labsheet_rows` and the `write_to_*` tools overwrite or add content to existing records. Review each change the agent proposes before you approve the tool call.

## Description

The Scispot MCP Server is a remote Model Context Protocol (MCP) server that integrates [Scispot](https://www.scispot.com)'s Lab Operating System with AI assistants like Claude. It provides 27 tools spanning Labsheet data management, Electronic Lab Notebook (ELN) operations, inventory/storage lookups, and Labflow sample tracking. The server runs as an AWS Lambda function and authenticates via OAuth 2.0 with PKCE.

## Features

- **Labsheet Management** — List, search, and filter Labsheet rows; inspect schemas; add and update data; organize by folders.
- **ELN Experiments** — List, read, create, and write content to experiments; link Labsheet rows for traceability.
- **ELN Protocols** — List, read, create, and write structured content (text, sections, files, images) to protocols; link Labsheet rows.
- **ELN Documentation** — List, read, create, and write to documentation entries; link Labsheet rows.
- **Storage & Inventory** — Browse storage locations (freezers, rooms, cabinets) and fetch manifests (plates, boxes, racks) by ID or barcode.
- **Labflows** — List sample UUIDs associated with a Labflow for downstream analysis.
- **Labspaces** — Discover all available Labspaces as the entry point for ELN navigation.

## Setup

### Claude web app (Anthropic MCP Directory)

Use these steps when using **Claude in the browser** (claude.ai), not Claude Code.

1. Visit the [Anthropic MCP Directory](https://claude.com/connectors)
2. Find and connect to **Scispot**
3. Complete the OAuth sign-in with your Scispot account
4. Start interacting with your workspace through Claude

### Claude Code (terminal)

Add the remote server, then sign in with OAuth:

```bash
claude mcp add scispot https://mcp.scispot.io/ --transport http
```

In Claude Code, run `/mcp`, select **scispot**, choose **Authenticate**, and sign in with your Scispot account.

#### Alternative: API key header

If your client can't complete the OAuth flow, you can pass a Scispot API key as a Bearer header instead:

```bash
claude mcp add scispot https://mcp.scispot.io/ --transport http --header "Authorization: Bearer YOUR_SCISPOT_API_KEY"
```

Replace `YOUR_SCISPOT_API_KEY` with your actual API key. Restart Claude Code or reload MCP configuration if prompted so the new server is picked up.

## Authentication

This server uses OAuth 2.0 with PKCE (S256). Clients that support remote MCP OAuth (Cursor, Claude, Claude Code) discover the authorization server from `https://mcp.scispot.io/.well-known/oauth-protected-resource`, register through dynamic client registration, and open the Scispot sign-in page. Sign in with your Scispot account to authorize the connection.

During the connection flow:

- If the authorization page asks for it, provide your **Scispot API key** as the client secret
- The API key is used solely as a Bearer token for Scispot API requests
- No credentials are persisted to permanent storage on the server
- Authorization codes expire within 5 minutes

You need a valid Scispot account with API access. Contact your Scispot administrator if you need an API key.

## Examples

### Example 1: Explore and query Labsheet data

**User prompt:** "Show me all the Labsheets in my workspace, then find the schema for the 'Cell Lines' Labsheet and search for rows where the Status column equals 'Active'."

**What happens:**
- Server calls `list_labsheets` to return all available Labsheets
- Server calls `get_labsheet_schema` for "Cell Lines" to discover column names and types
- Server calls `search_labsheet_rows` with a mandatory filter on the Status column
- Returns matching rows with all column data

### Example 2: Create an experiment and write a protocol summary

**User prompt:** "Create a new experiment called 'Western Blot Optimization Round 3' in the 'Protein Lab' Labspace, then write a summary of the protocol steps I'll be using."

**What happens:**
- Server calls `list_labspaces` to confirm "Protein Lab" exists and get its location
- Server calls `create_experiment_stub` with the name and location
- Server calls `write_to_experiment` to append the protocol summary as HTML content
- Returns confirmation with the new experiment details

### Example 3: Link Labsheet samples to a protocol with structured content

**User prompt:** "Find all samples in the 'Reagent Inventory' Labsheet where the Lot Number is 'LOT-2025-001', link them to the 'Sample Preparation v2' protocol in the 'QC Lab' Labspace, and add a structured section to the protocol documenting the linked samples."

**What happens:**
- Server calls `search_labsheet_rows` to find matching rows in "Reagent Inventory"
- Server calls `link_labsheet_to_protocol` to create traceability links between the rows and the protocol
- Server calls `write_to_protocol` with `content_type='json'` and a structured elements array containing a section header and text descriptions
- Returns confirmation of linked rows and updated protocol content

### Example 4: Browse storage locations and fetch a manifest

**User prompt:** "Show me all storage locations in my workspace, then look up the manifest with barcode 'PLT-00142'."

**What happens:**
- Server calls `list_locations` to return all root-level storage locations (freezers, rooms, cabinets)
- Server calls `get_manifest` with the barcode to fetch the plate/container details including all wells or slots
- Returns the full manifest layout with sample positions

### Example 5: Navigate ELN structure and read documentation

**User prompt:** "List all Labspaces, then show me the protocols in the 'Genomics' Labspace, and read the full content of the 'DNA Extraction SOP' protocol."

**What happens:**
- Server calls `list_labspaces` to discover available Labspaces
- Server calls `list_protocols` with location "Genomics" to list all protocols
- Server calls `get_protocol` by name to fetch the full protocol content
- Returns the complete protocol text with formatting

## Available Tools

### Read-Only Tools (16)
| Tool | Description |
|------|-------------|
| `list_labsheets` | List all Labsheets in the workspace |
| `get_labsheet_schema` | Get column definitions for a Labsheet |
| `search_labsheet_rows` | Search and filter Labsheet rows |
| `get_labsheet_row` | Fetch a single row by UUID |
| `list_labsheet_folders` | List folders or Labsheets within a folder |
| `list_labspaces` | List all ELN Labspaces |
| `list_experiments` | List experiments in a location |
| `get_experiment` | Fetch full experiment content |
| `list_protocols` | List protocols in a location |
| `get_protocol` | Fetch full protocol content |
| `list_documentations` | List documentation entries in a location |
| `get_documentation` | Fetch full documentation content |
| `list_locations` | List root-level storage locations |
| `get_manifest` | Fetch a manifest by hrid, uuid, or barcode |
| `list_labflow_samples` | List sample UUIDs for a Labflow |
| `get_image_thumbnail` | Download a row's image thumbnail |

### Write Tools (11)
| Tool | Destructive | Description |
|------|-------------|-------------|
| `add_labsheet_rows` | No | Add new rows to a Labsheet |
| `update_labsheet_rows` | Yes | Overwrite existing row values |
| `create_experiment_stub` | No | Create an empty experiment |
| `write_to_experiment` | Yes | Append/prepend content to an experiment |
| `create_protocol_stub` | No | Create an empty protocol |
| `write_to_protocol` | Yes | Write text, structured JSON, or images to a protocol |
| `create_documentation_stub` | No | Create an empty documentation entry |
| `write_to_documentation` | Yes | Append/prepend content to documentation |
| `link_labsheet_to_experiment` | No | Link Labsheet rows to an experiment |
| `link_labsheet_to_protocol` | No | Link Labsheet rows to a protocol |
| `link_labsheet_to_documentation` | No | Link Labsheet rows to documentation |

## Privacy Policy

This MCP server processes only the data necessary to execute requested operations against your Scispot workspace. No conversation data is collected or stored.

For complete privacy information, see our privacy policy: https://www.scispot.com/legal/privacy-policy

## Support

- **Email:** team@scispot.io
- **API Documentation:** https://docs.scispot.com/
- **Website:** https://www.scispot.com