---
name: appsentinels
description: "AppSentinels API Security Platform CLI and ATDv2 rule authoring. Use when working with (1) as-mcp-cli commands for tenant discovery, API catalog, security events, tags management, (2) ATDv2 threat detection rules - creating, validating, deploying YAML-based detection rules, (3) API security analysis and threat pattern identification, (4) Coverage analysis for detection rules. Triggers on create ATDv2 rule, list APIs, tenant, security events, api tags, threat detection, as-mcp-cli, validate rule, enable rule."
---

# AppSentinels API Security Platform

## Overview

AppSentinels provides API security through the MCP CLI (`as-mcp-cli`) and ATDv2 threat detection engine.

## Installation & Setup

### Install as-mcp-cli
```bash
# From source
cd /path/to/as-mcp-cli
pip install --user .

# Or via pip (if published)
pip install as-mcp-cli
```

### Quick Start
```bash
# 1. Add MCP server and authenticate (opens browser for OAuth)
as-mcp-cli add appsentinels https://your-mcp-server.com/mcp/sse

# 2. Verify setup
as-mcp-cli list                                    # Show configured servers
as-mcp-cli mcp appsentinels tenant all-tenants     # Test connection

# 3. Re-authenticate if token expires
as-mcp-cli auth appsentinels --force
```

### as-mcp-cli Commands
| Command | Description |
|---------|-------------|
| `add <name> <url>` | Add new MCP server and authenticate |
| `mcp <name> <command>` | Run command on MCP server |
| `auth <name> --force` | Re-authenticate with server |
| `list` | List configured servers |
| `remove <name>` | Remove server |

**Credentials**: Stored in `~/.claude/.credentials.json`

## CLI Usage

All commands use: `as-mcp-cli mcp appsentinels <command>`

### Quick Start
```bash
tenant all-tenants                    # List all tenants
api list <tenant_id>                  # List APIs
api tags list <tenant_id>             # List tags
api info <tenant_id> <api_id>         # API details
```

### Command Categories

| Category | Commands |
|----------|----------|
| `tenant` | `organizations`, `tenants --org-id <id>`, `all-tenants` |
| `api` | `list`, `info`, `tags list`, `tags apis`, `tags create`, `tags assign` |
| `atdv2` | `health check`, `rules list/get/create/update/delete/enable/disable/validate`, `jobs list/status` |
| `security-events` | `list`, `info`, `summary`, `update`, `delete` |

### API List Filters
```bash
api list <tid> --risk critical,high   # By risk level
api list <tid> --sensitive            # Sensitive data APIs
api list <tid> --shadow               # Shadow APIs
api list <tid> --methods POST,PUT     # By HTTP method
api list <tid> --tags tag1,tag2       # By tags
api list <tid> --uri-pattern "/admin" # Path matching
```

### Tag Operations
```bash
api tags list <tid>                           # List all tags
api tags apis <tid> --tag-id <id>             # APIs with tag
api tags create <tid> "name" --description "" # Create tag
api tags assign <tid> --tag "name" --api-ids 1,2,3  # Assign tag
```

## ATDv2 Rule Creation

### Pipeline
```
raw_data -> PRE-FILTER -> EXTRACTION -> NORMALIZE -> POST-FILTER -> DETECTION -> EVIDENCE -> ALERTS
```

### Stage Prefixes (Required)
| Stage | Prefix | Type |
|-------|--------|------|
| Pre-filter | `prefilter_` | `pre_filter` |
| Extraction | `extraction_` | `extraction` |
| Normalize | `normalize_` | `normalize` |
| Post-filter | `filter_` | `filter` |
| Detection | `detection_` | `detection` |
| Evidence | `evidence_` | `evidence` |
| Alert | `alerts_` | `alert` |

### Rule Template
```yaml
rule:
  name: "Rule-Name"
  engine: atdv2
  description: "Description"
  duration: "15m"
  frequency: "5m"

  prefilter_stage:
    type: pre_filter
    source: raw_data
    filters:
      - field: APIID
        operator: IN
        value: '@tags:tag_name'

  extraction_stage:
    type: extraction
    source: prefilter_stage
    fields:
      - source: HTTPReq_Body
        path: field_name
        alias: extracted_field

  filter_stage:
    type: filter
    source: extraction_stage
    filters:
      - field: extracted_field
        operator: Exists
        value: True

  detection_stage:
    type: detection
    source: filter_stage
    group_by: _ip
    metric: distinct_count
    field: extracted_field
    threshold: 5
    operator: GT

  evidence_stage:
    type: evidence
    source: detection_stage
    limit: 20
    fields:
      - extracted_field
      - _ip

  alerts_stage:
    type: alert
    source: detection_stage
    category: "Automated Threat"
    subcategory: "Threat Type"
    subject: groupby_field
    template: true
    message: "Detected from {groupby_field} with {column:extracted_field}"
    severity: critical
    title: "Alert Title"
```

### Key Gotchas
- Alert stage: prefix `alerts_` but type is `alert`
- Set `template: true` when using `{column:field}` or `{groupby_field}`
- URL path extraction uses 1-based indexing
- Use `@tags:tag_name` for dynamic API filtering

### Filter Operators
| Operator | Usage |
|----------|-------|
| `EQ`, `NEQ` | Equality |
| `GT`, `GTE`, `LT`, `LTE` | Comparison |
| `IN`, `NOT_IN` | Set membership |
| `LIKE`, `NOT_LIKE` | Pattern match |
| `Exists`, `NotExists` | Field presence |

### Detection Metrics
| Metric | Use Case |
|--------|----------|
| `count` | Total events (low fidelity) |
| `distinct_count` | Unique values (high fidelity) |
| `sum`, `avg`, `max`, `min` | Aggregations |

### Alert Template Variables
- `{groupby_field}` - Grouped field value
- `{detection_count}` - Event count
- `{column:field_name}` - Evidence column values
- `{column:field_name_count}` - Unique value count
- `{first_seen}`, `{last_seen}` - Timestamps

## Rule Workflow

```bash
# 1. Discover APIs and tags
api tags list <tenant_id>
api tags apis <tenant_id> --tag-id <id>
api info <tenant_id> <api_id>

# 2. Validate rule
atdv2 rules validate <tenant_id> --content "<yaml>"

# 3. Create and enable
atdv2 rules create <tenant_id> <name> --content "<yaml>"
atdv2 rules enable <tenant_id> <name>

# 4. Monitor
atdv2 jobs status <tenant_id> <name>
security-events list <tenant_id> --limit 50
```

## Reference Files

- **Installation Guide**: See [references/installation.md](references/installation.md) for detailed setup instructions
- **ATDv2 DSL Reference**: See [references/atdv2-dsl.md](references/atdv2-dsl.md) for complete DSL syntax
- **CLI Commands**: See [references/cli-commands.md](references/cli-commands.md) for full command reference
- **Rule Workflow**: See [references/rule-workflow.md](references/rule-workflow.md) for detailed creation flow
