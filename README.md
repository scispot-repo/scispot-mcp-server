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

Start with read-only requests. Review the data and the planned action before you approve a write.

For important records, ask the agent to summarize the proposed changes before it creates or updates data. Review the resulting record in Scispot after the action completes.

Eight tools are marked destructive because they overwrite existing rows or page content: `update_labsheet_rows`, `update_labsheet_rows_by_id_column`, `write_to_experiment`, `write_to_protocol`, `write_to_documentation`, and `link_manifest_to_*` (when `position` is `replace`).

For regulated workflows, make sure your use of Cursor and its data handling meets your organization's validation, review, and approval requirements.

## Description

The Scispot MCP Server is a remote Model Context Protocol (MCP) server that integrates [Scispot](https://www.scispot.com)'s Lab Operating System with AI assistants like Claude. It provides 55 tools spanning Labsheet data management, Electronic Lab Notebook (ELN) operations, inventory/storage lookups, Labflow sample tracking, and Jazz apps. The server runs as an AWS Lambda function and authenticates via OAuth 2.0 with PKCE.

## Features

- **Labsheet Management** — List, search, and filter Labsheet rows; inspect schemas; add and update data; organize by folders.
- **ELN Experiments** — List, read, create, and write content to experiments; link Labsheet rows for traceability.
- **ELN Protocols** — List, read, create, and write structured content (text, sections, files, images) to protocols; link Labsheet rows.
- **ELN Documentation** — List, read, create, and write to documentation entries; link Labsheet rows.
- **Storage & Inventory** — Browse storage locations (freezers, rooms, cabinets) and fetch manifests (plates, boxes, racks) by ID or barcode.
- **Labflows** — List sample UUIDs associated with a Labflow for downstream analysis.
- **Labspaces** — Discover Labspaces and folders, and create folders for ELN pages.
- **Tags** — List tag categories, search tag values, and set tags on ELN pages.
- **Jazz apps** — Read the build guide, then list, create, upload, publish, and roll back custom Jazz apps.

## Setup

### Choose MCP or the REST API

Use the MCP Server when you want an AI assistant to work with Scispot conversationally. Use the REST API when you build a programmatic integration or automation.

### Before you begin

You need:

- A Super Admin Scispot account, or an account with Dry Lab access
- An MCP-compatible AI client

The MCP Server uses `https://mcp.scispot.io`.

### Security and permissions

The MCP Server respects your existing Scispot permissions. You can access only the data and actions that your Scispot account allows.

Web connectors use OAuth. Sign in with your Scispot account when the client asks you to authorize the connection.

### Connect Claude.ai

Use Claude.ai's custom connector flow. Sign in with your Scispot account when prompted.

1. In Claude.ai, open **Settings**, then **Connectors**. Select **Add custom connector**.
2. Search for **Scispot.io** in the connector search bar.
3. Click **Connect to Claude**.
4. Complete authentication by logging in with your Scispot account.

Test the connection with a read-only request such as: "List my Scispot Labsheets."

### Connect Claude Code

Claude Code supports remote MCP servers over HTTP. Add Scispot as an HTTP MCP server, then authenticate in the client.

1. In your terminal, run:

```bash
claude mcp add --transport http scispot-mcp https://mcp.scispot.io/
```

2. Open Claude Code with the `claude` command.
3. Enter `/mcp` to check your MCP connections.
4. Select the **scispot-mcp** connection, choose **Authenticate**, and log in with your Scispot account.
5. Start or restart Claude Code. Ask it to list your Scispot Labsheets to confirm that it can discover the server tools.

Test the connection with a read-only request such as: "List my Scispot Labsheets."

### Connect another MCP-compatible client

You can connect other clients that support remote HTTP MCP servers. The exact labels vary by client.

1. Add a remote MCP server or connector in your client.
2. Set the connection URL to `https://mcp.scispot.io`.
3. Choose OAuth when your client supports the Scispot OAuth flow.
4. When prompted, log in with your Scispot account.
5. Confirm the connection with a read-only request before you perform a write operation.

Contact Customer Success at [cs@scispot.io](mailto:cs@scispot.io) if your client cannot complete the OAuth flow or does not support custom HTTP headers.

### Use write actions safely

Start with read-only requests. Review the data and the planned action before you approve a write.

For important records, ask the AI to summarize the proposed changes before it creates or updates data. Review the resulting record in Scispot after the action completes.

Tools that change data require the same permissions as the equivalent action in Scispot. Eight tools are marked destructive because they overwrite existing rows or page content: `update_labsheet_rows`, `update_labsheet_rows_by_id_column`, `write_to_experiment`, `write_to_protocol`, `write_to_documentation`, and `link_manifest_to_*` (when `position` is `replace`).

For regulated workflows, make sure your use of the AI client and its data handling meets your organization's validation, review, and approval requirements.

## Troubleshooting

### You cannot add the connector

Verify that you entered `https://mcp.scispot.io` exactly.

### Authentication fails

Contact Customer Success at [cs@scispot.io](mailto:cs@scispot.io) if you cannot authenticate.

### You cannot see data or perform an action

The MCP Server uses your Scispot permissions. Confirm that you can complete the same action in Scispot with your account.

### The AI proposes an unexpected change

Do not approve the action. Ask the AI to explain the proposed change, then verify the target records in Scispot before you continue.

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

Tools below match the production server at `https://mcp.scispot.io/` (55 tools). Destructive follows the server's `destructiveHint`: those calls can overwrite existing rows or page content.

### Read-Only Tools (27)
| Tool | Description |
|------|-------------|
| `list_labsheets` | List all Labsheets in the workspace |
| `get_labsheet_schema` | Get column definitions for a Labsheet |
| `search_labsheet_rows` | Search and filter Labsheet rows |
| `get_labsheet_row` | Fetch a single row by UUID |
| `get_labsheet_row_by_id` | Fetch a single row by its ID column value |
| `list_labsheet_folders` | List Labsheet folders, or the Labsheets in a folder |
| `get_image_thumbnail` | Download a row's image thumbnail |
| `list_labspaces` | List all ELN Labspaces |
| `get_labspace` | Fetch a Labspace by name |
| `get_labspace_folder` | Fetch an ELN folder by name or UUID |
| `list_experiments` | List experiments in a folder or location |
| `list_experiment_page_templates` | List experiment templates available when creating an experiment |
| `get_experiment` | Fetch full experiment content |
| `list_protocols` | List protocols in a folder or location |
| `list_protocol_page_templates` | List protocol templates available when creating a protocol |
| `get_protocol` | Fetch full protocol content |
| `list_documentations` | List documentation entries in a folder or location |
| `list_documentation_page_templates` | List documentation templates available when creating an entry |
| `get_documentation` | Fetch full documentation content |
| `list_tag_categories` | List tag categories used on ELN pages and templates |
| `search_tags` | Search tag values across categories |
| `list_locations` | List root-level storage locations |
| `get_manifest` | Fetch a manifest by hrid, uuid, or barcode |
| `list_labflow_samples` | List sample UUIDs for a Labflow |
| `get_jazz_guide` | Read the guide for building a Jazz app |
| `list_jazz_apps` | List Jazz apps in the workspace |
| `list_jazz_app_versions` | List versions of a Jazz app |

### Write Tools (28)
| Tool | Destructive | Description |
|------|-------------|-------------|
| `create_labsheet` | No | Create a Labsheet with columns |
| `create_labsheet_column` | No | Add a column to a Labsheet |
| `rename_labsheet_column` | No | Rename a Labsheet column |
| `add_labsheet_rows` | No | Add new rows to a Labsheet |
| `update_labsheet_rows` | Yes | Overwrite existing row values |
| `update_labsheet_rows_by_id_column` | Yes | Update existing rows by their ID column |
| `create_labspace_folder` | No | Create an ELN folder under a Labspace or parent folder |
| `create_experiment_stub` | No | Create an experiment, optionally from a template |
| `write_to_experiment` | Yes | Write text, structured JSON, or images to an experiment |
| `link_labsheet_to_experiment` | No | Link Labsheet rows to an experiment |
| `link_manifest_to_experiment` | Yes | Link a manifest to an experiment; `position=replace` can overwrite the page |
| `create_protocol_stub` | No | Create a protocol, optionally from a template |
| `write_to_protocol` | Yes | Write text, structured JSON, or images to a protocol |
| `link_labsheet_to_protocol` | No | Link Labsheet rows to a protocol |
| `link_manifest_to_protocol` | Yes | Link a manifest to a protocol; `position=replace` can overwrite the page |
| `create_documentation_stub` | No | Create a documentation entry, optionally from a template |
| `write_to_documentation` | Yes | Write text, structured JSON, or images to documentation |
| `link_labsheet_to_documentation` | No | Link Labsheet rows to documentation |
| `link_manifest_to_documentation` | Yes | Link a manifest to documentation; `position=replace` can overwrite the page |
| `upload_labspace_image` | No | Upload an image (up to 5 MiB) for an ELN page |
| `request_large_file_upload` | No | Prepare an upload for a file between 5 MiB and 100 MiB |
| `attach_uploaded_image_to_labspace_page` | No | Embed an uploaded image in an experiment, protocol, or documentation page |
| `set_page_tags` | No | Replace the tags on an ELN page |
| `create_jazz_app` | No | Create a Jazz app and return its upload form |
| `upload_jazz_app_content` | No | Return a presigned form for uploading built Jazz app files |
| `publish_jazz_app_version` | No | Make an uploaded Jazz app version the live one |
| `rollback_jazz_app` | No | Point a Jazz app back at a previously published version |
| `create_jazz_dev_session` | No | Mint a short-lived session for exercising a Jazz app |

## Privacy Policy

This MCP server processes only the data necessary to execute requested operations against your Scispot workspace. No conversation data is collected or stored.

For complete privacy information, see our privacy policy: https://www.scispot.com/legal/privacy-policy

## Support

- **Email:** team@scispot.io
- **API Documentation:** https://docs.scispot.com/
- **Website:** https://www.scispot.com