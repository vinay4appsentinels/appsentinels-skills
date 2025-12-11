# ATDv2 Rule Creation Workflow

## Workflow Overview

```
DISCOVERY -> DESIGN -> WRITE -> VALIDATE -> DEPLOY -> MONITOR
```

---

## Step 1: Discovery

### 1.1 Identify Target APIs

Start with tags - they determine which APIs your rule monitors.

```bash
# List all tags
api tags list <tenant_id>

# Check APIs with specific tag
api tags apis <tenant_id> --tag-id <tag_id>
```

**Important**: A rule using a tag with 0 APIs will never trigger.

### 1.2 Analyze API Parameters

```bash
api info <tenant_id> <api_id> --format json
```

**Key fields in API info:**

| Field | What to Look For | Extraction Source |
|-------|------------------|-------------------|
| `request_body_2xx` | JSON body parameters | `HTTPReq_Body` |
| `request_body_pii` | PII sample payload | `HTTPReq_Body` |
| `request_query` | Query parameters | `HTTPReq_Query` |
| `uri_path_parameter` | Path parameters | `HTTPReq_Uri` (index) |

### 1.3 Map Parameters to Threats

| Parameter Type | Examples | Detection |
|----------------|----------|-----------|
| Credentials | username, email, password, otp | `distinct_count` on field |
| Financial | card_number, coupon_code | `distinct_count` on field |
| Identity | user_id, customer_id | `distinct_count` on field |
| Resource | file_id, order_id | `distinct_count` on field |

---

## Step 2: Design

### Choose Detection Fidelity

| Fidelity | Metric | Use Case |
|----------|--------|----------|
| Low | `count` | Volume detection, higher false positives |
| High | `distinct_count` | Pattern detection, lower false positives |

### Choose Time Window

| Duration | Frequency | Use Case |
|----------|-----------|----------|
| 5m | 1m | Real-time attacks |
| 15m | 5m | Standard detection |
| 1h | 15m | Slow attacks |
| 24h | 1h | Long-running patterns |

### Document Design

| Item | Value |
|------|-------|
| Tag | Tag name and API count |
| Key Parameters | Fields to extract |
| Attack Type | Brute force, enumeration, etc. |
| Detection Field | Parameter for `distinct_count` |
| Group By | `_ip`, `session_id`, etc. |
| Threshold | Starting value |

---

## Step 3: Write Rule

### Minimal Rule Template

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
    message: "Detected {column:extracted_field_count} values from {groupby_field}"
    severity: critical
    title: "Alert Title"
```

---

## Step 4: Validate

### Validation Loop

```bash
# Validate
atdv2 rules validate <tenant_id> --content "<yaml>"

# If errors, fix and re-validate
# Repeat until "Validation successful"
```

### Common Validation Errors

| Error | Fix |
|-------|-----|
| Invalid stage prefix | Rename stage (e.g., `filter_x` not `post_filter_x`) |
| Unknown source reference | Check spelling, ensure source stage exists |
| Missing required field | Add `metric`, `threshold`, `operator` |
| Invalid operator | Use valid operator from reference |
| Tag not found | Verify tag exists: `api tags list` |

---

## Step 5: Deploy

```bash
# Check if rule name exists
atdv2 rules exists <tenant_id> <rule_name>

# Create rule (disabled by default)
atdv2 rules create <tenant_id> <rule_name> --content "<yaml>"

# Enable rule
atdv2 rules enable <tenant_id> <rule_name>

# Verify
atdv2 rules get <tenant_id> <rule_name>
```

---

## Step 6: Monitor

### Check Job Status

```bash
# List all jobs
atdv2 jobs list <tenant_id>

# Check specific rule
atdv2 jobs status <tenant_id> <rule_name>
```

### Review Security Events

```bash
security-events list <tenant_id> --limit 50
security-events summary <tenant_id> --time-range 7d
```

### Tune the Rule

| Symptom | Action |
|---------|--------|
| Too many alerts | Increase threshold, add filters |
| No alerts | Decrease threshold, broaden filters |
| Missing context | Add more evidence fields |

### Update or Disable

```bash
# Update
atdv2 rules update <tenant_id> <rule_name> --content "<yaml>"

# Disable
atdv2 rules disable <tenant_id> <rule_name>

# Delete
atdv2 rules delete <tenant_id> <rule_name>
```

---

## Coverage Analysis

### 3-Layer Coverage Check

1. **Tag exists**: Does the tag have APIs?
2. **APIs tagged**: Are relevant APIs assigned to tag?
3. **Parameters match**: Do APIs have the parameters the rule extracts?

### True Coverage Script

```python
# For each tagged API:
# 1. Get API info
# 2. Check if expected parameters exist in request_body_2xx/4xx
# 3. Calculate: truly_covered / total_tagged = true_coverage_pct
```

### Common Coverage Issues

| Issue | Cause | Fix |
|-------|-------|-----|
| Low coverage | APIs don't have expected param | Review API params, add alternate extractions |
| 0% coverage | Tag doesn't exist | Create tag and assign APIs |
| Rule extracts wrong field | API uses different param name | Update extraction path |
