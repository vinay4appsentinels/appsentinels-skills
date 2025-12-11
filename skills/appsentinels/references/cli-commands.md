# AppSentinels CLI Command Reference

## Table of Contents
- [Tenant Management](#tenant-management)
- [API Catalog](#api-catalog)
- [API Tags](#api-tags)
- [ATDv2 Operations](#atdv2-operations)
- [Security Events](#security-events)
- [Settings Management](#settings-management)

## Command Format

All commands use: `as-mcp-cli mcp appsentinels <command> [options]`

Common options:
- `--format table|json` - Output format
- `--limit N` - Number of results
- `--page N` - Page number (0-indexed)

---

## Tenant Management

### List Organizations
```bash
tenant organizations [--format table|json]
tenant orgs  # alias
```

### List Tenants in Organization
```bash
tenant tenants --org-id <org_id> [--format table|json]
```

### List All Tenants
```bash
tenant all-tenants [--format table|json]
```

---

## API Catalog

### List APIs
```bash
api list <tenant_id> [options]
```

**Basic Options:**
- `--limit N` - Results per page (default: 10, max: 100)
- `--page N` - Page number (starts from 0)
- `--sort-by FIELD` - Sort by: risk, discovered, observed, method
- `--desc` - Sort descending

**Security Filters:**
- `--risk LEVELS` - Risk levels: critical, high, medium, low (comma-separated)
- `--shadow` - Shadow/undocumented APIs
- `--sensitive` - APIs with sensitive data
- `--public` - Public-facing APIs
- `--internal` - Internal-only APIs
- `--no-auth` - APIs without authentication
- `--privileged` - APIs requiring elevated access

**Operational Filters:**
- `--new` - Recently discovered APIs
- `--unused` - APIs with no recent traffic
- `--methods LIST` - HTTP methods: GET, POST, PUT, DELETE (comma-separated)
- `--path PATTERN` - URL path pattern matching
- `--tags LIST` - Filter by API tags (comma-separated)

**Examples:**
```bash
api list tenant_id --risk critical,high --limit 20
api list tenant_id --shadow --new
api list tenant_id --public --no-auth --sensitive
api list tenant_id --methods POST,PUT,DELETE --path "/api/users"
```

### Get API Details
```bash
api info <tenant_id> <api_id> [--format json]
```

Returns:
- Security risk assessment
- Authentication requirements
- Parameter analysis (request_body_2xx, request_body_4xx, request_query)
- Vulnerability information
- Traffic patterns
- Sample curl commands (curl_2xx, curl_4xx)

---

## API Tags

### List Tags
```bash
api tags list <tenant_id> [--limit N] [--format json]
```

Returns tag id, name, and endpoint_count for each tag.

### Get APIs with Tag
```bash
api tags apis <tenant_id> --tag-id <tag_id> [--format json]
```

Returns list of APIs. Filter for `tagged: true` to get actually tagged APIs.

### Create Tag
```bash
api tags create <tenant_id> "tag_name" --description "Description"
```

### Assign Tag to APIs
```bash
api tags assign <tenant_id> --tag "tag_name" --api-ids 1,2,3
```

---

## ATDv2 Operations

### Health Check
```bash
atdv2 health check                    # Service health
atdv2 health status <tenant_id>       # Tenant ATDv2 status
```

### Rule Management

**List Rules:**
```bash
atdv2 rules list <tenant_id> [--format json]
```

**Get Rule:**
```bash
atdv2 rules get <tenant_id> <rule_name> [--format json|yaml]
```

**Create Rule:**
```bash
atdv2 rules create <tenant_id> <rule_name> --content "<yaml>"
atdv2 rules create <tenant_id> <rule_name> --file rule.yaml
```

**Update Rule:**
```bash
atdv2 rules update <tenant_id> <rule_name> --content "<yaml>"
atdv2 rules update <tenant_id> <rule_name> --file rule.yaml
```

**Delete Rule:**
```bash
atdv2 rules delete <tenant_id> <rule_name> [--force]
```

**Enable/Disable:**
```bash
atdv2 rules enable <tenant_id> <rule_name>
atdv2 rules disable <tenant_id> <rule_name>
```

**Validate Rule:**
```bash
atdv2 rules validate <tenant_id> --content "<yaml>"
atdv2 rules validate <tenant_id> --file rule.yaml
```

**Check if Rule Exists:**
```bash
atdv2 rules exists <tenant_id> <rule_name>
```

### Job Monitoring
```bash
atdv2 jobs list <tenant_id>                    # List all active jobs
atdv2 jobs status <tenant_id> <rule_name>      # Job execution status
```

---

## Security Events

### List Events
```bash
security-events list <tenant_id> [options]
```

**Filters:**
- `--severity LEVELS` - critical, high, medium, low
- `--status STATUS` - open, investigating, resolved
- `--category CAT` - Event category
- `--subcategory SUBCAT` - Event subcategory
- `--from-date DATE` - Start date
- `--to-date DATE` - End date
- `--limit N` - Number of results

### Get Event Details
```bash
security-events info <tenant_id> <event_id> [--include-children]
```

### Event Summary
```bash
security-events summary <tenant_id> [--time-range 7d]
```

### Update Event
```bash
security-events update <tenant_id> <event_id> --status resolved
```

### Delete Events
```bash
security-events delete <tenant_id> <event_ids> [--force] [--reason "reason"]
```

---

## Settings Management

### Organization Settings

**User Management:**
```bash
settings org users list [--search TERM]
settings org users create --user-id USER --role ROLE
settings org users update --user-id USER --role ROLE
settings org users delete --user-id USER
settings org users activate --user-id USER
settings org users deactivate --user-id USER
```

**Application Management:**
```bash
settings org apps list
settings org apps create --name NAME --description DESC
settings org apps update --id APP_ID --name NAME
settings org apps delete --id APP_ID
settings org apps enable --id APP_ID
settings org apps disable --id APP_ID
```

**RBAC:**
```bash
settings org rbac list
settings org rbac create --id ROLE_ID --title TITLE --permissions PERMS
settings org rbac assign --user-id USER --role-id ROLE
```

### Security Settings

**Threat Actors:**
```bash
settings security threat-actors get
settings security threat-actors update --config-file FILE
```

**Risk Levels:**
```bash
settings security risk-levels get
settings security risk-levels update --config-file FILE
```

**ATD Profiles:**
```bash
settings security atd list
settings security atd get --profile PROFILE --type TYPE
settings security atd enable --profile PROFILE
settings security atd disable --profile PROFILE
```
