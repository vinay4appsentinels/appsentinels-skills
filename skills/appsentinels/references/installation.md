# as-mcp-cli Installation Guide

## Overview

`as-mcp-cli` is a CLI tool for interacting with MCP (Model Context Protocol) servers. It connects to AppSentinels MCP server using SSE (Server-Sent Events) protocol and OAuth authentication.

## Installation

### From Source

```bash
cd /path/to/as-mcp-cli
pip install --user .
```

### Via pip (if published)

```bash
pip install as-mcp-cli
```

## Quick Start

### 1. Add an MCP Server

```bash
as-mcp-cli add appsentinels https://your-mcp-server.com/mcp/sse
```

This command:
- Registers the server with the name "appsentinels"
- Initiates OAuth authentication (opens browser)
- Stores credentials in `~/.claude/.credentials.json`

### 2. Verify Setup

```bash
# List configured servers
as-mcp-cli list

# Test connection
as-mcp-cli mcp appsentinels tenant all-tenants
```

### 3. Start Using Commands

```bash
as-mcp-cli mcp appsentinels api list <tenant_id>
as-mcp-cli mcp appsentinels api tags list <tenant_id>
as-mcp-cli mcp appsentinels atdv2 rules list <tenant_id>
```

---

## Commands Reference

### Core Commands

| Command | Description |
|---------|-------------|
| `mcp <name> <command>` | Run a command on an MCP server |
| `add <name> <url>` | Add a new MCP server and authenticate |
| `auth <name> [options]` | Re-authenticate with an MCP server |
| `list` | List configured MCP servers |
| `remove <name>` | Remove an MCP server |

### `add` - Add New Server

```bash
as-mcp-cli add <name> <server-url> [--client-id ID]
```

**Arguments:**
- `name` - Name for the MCP server (e.g., "appsentinels")
- `server-url` - MCP server SSE URL

**Options:**
- `--client-id ID` - OAuth client ID (optional)

**Examples:**
```bash
as-mcp-cli add appsentinels https://mcp.appsentinels.ai/mcp/sse
as-mcp-cli add my-server https://example.com/mcp/sse --client-id abc123
```

### `mcp` - Run Commands

```bash
as-mcp-cli mcp <name> [--debug] <command>
```

**Options:**
- `--debug` - Enable debug output (shows SSE events)

**Examples:**
```bash
as-mcp-cli mcp appsentinels tenant all-tenants
as-mcp-cli mcp appsentinels api list nykaa_production --limit 10
as-mcp-cli mcp appsentinels --debug api tags list nykaa_production
```

### `auth` - Re-authenticate

```bash
as-mcp-cli auth <name> [options]
```

**Options:**
- `--server-url URL` - MCP server SSE URL (required for new servers)
- `--client-id ID` - OAuth client ID (optional)
- `--force` - Force re-authentication even if token is valid

**Examples:**
```bash
as-mcp-cli auth appsentinels --force
as-mcp-cli auth my-server --server-url https://example.com/mcp/sse
```

### `list` - List Servers

```bash
as-mcp-cli list
```

Shows all configured servers with:
- Server name
- SSE URL
- Token status (valid, expired, time remaining)

### `remove` - Remove Server

```bash
as-mcp-cli remove <name>
```

Removes server configuration from credentials file.

---

## Authentication

### OAuth 2.0 Flow

The CLI uses OAuth 2.0 with PKCE (Proof Key for Code Exchange):

1. Discovers OAuth endpoints via `.well-known/oauth-authorization-server`
2. Opens browser for user authentication
3. Receives callback with authorization code
4. Exchanges code for access and refresh tokens
5. Stores credentials in `~/.claude/.credentials.json`

### Token Management

- Tokens are automatically refreshed when expired (if refresh token available)
- Use `--force` flag to manually re-authenticate
- Token status visible via `as-mcp-cli list`

### Credentials Storage

Credentials are stored at: `~/.claude/.credentials.json`

---

## Typical Workflow

```bash
# 1. Add the AppSentinels MCP server
as-mcp-cli add appsentinels https://mcp.appsentinels.ai/mcp/sse

# 2. List available tenants
as-mcp-cli mcp appsentinels tenant all-tenants

# 3. Explore APIs
as-mcp-cli mcp appsentinels api list <tenant_id> --limit 20
as-mcp-cli mcp appsentinels api tags list <tenant_id>

# 4. Work with ATDv2 rules
as-mcp-cli mcp appsentinels atdv2 rules list <tenant_id>
as-mcp-cli mcp appsentinels atdv2 rules validate <tenant_id> --content "<yaml>"

# 5. Check security events
as-mcp-cli mcp appsentinels security-events list <tenant_id> --limit 50
```

---

## Troubleshooting

### Token Expired

```bash
as-mcp-cli auth appsentinels --force
```

### Server Not Found

```bash
# Check configured servers
as-mcp-cli list

# Re-add the server
as-mcp-cli add appsentinels https://mcp.appsentinels.ai/mcp/sse
```

### Debug Mode

Enable debug output to see SSE protocol details:

```bash
as-mcp-cli mcp appsentinels --debug tenant all-tenants
```

### Connection Issues

1. Verify the server URL is correct
2. Check network connectivity
3. Ensure OAuth credentials are valid
4. Try re-authenticating with `--force`
