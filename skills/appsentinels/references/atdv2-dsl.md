# ATDv2 Rule DSL Reference

## Table of Contents
- [Rule Structure](#rule-structure)
- [Pipeline Stages](#pipeline-stages)
- [Field Extraction](#field-extraction)
- [Filter Operators](#filter-operators)
- [Detection Configuration](#detection-configuration)
- [Alert Templates](#alert-templates)
- [Common Patterns](#common-patterns)
- [Troubleshooting](#troubleshooting)

---

## Rule Structure

### Required Fields
```yaml
rule:
  name: "Rule-Name"           # Unique identifier
  engine: atdv2               # Must be "atdv2"
  description: "Description"  # Human-readable description
  duration: "15m"             # Time window: 5m, 15m, 1h, 24h
  frequency: "5m"             # Execution interval
```

### Stage Naming Convention

| Stage Type | Required Prefix | Type Value |
|------------|----------------|------------|
| Pre-filter | `prefilter_` | `pre_filter` |
| Extraction | `extraction_` | `extraction` |
| Normalize | `normalize_` | `normalize` |
| Post-filter | `filter_` | `filter` |
| Detection | `detection_` | `detection` |
| Evidence | `evidence_` | `evidence` |
| Alert | `alerts_` | `alert` |

**Important**: Alert stage uses prefix `alerts_` but type is `alert`.

---

## Pipeline Stages

### 1. Pre-filter Stage
Reduces data volume early in pipeline.

```yaml
prefilter_stage:
  type: pre_filter
  source: raw_data
  filters:
    - field: APIID
      operator: IN
      value: '@tags:tag_name'
    - field: HTTPResp_ResponseCode
      operator: GTE
      value: 200
```

### 2. Extraction Stage
Extracts values from JSON fields or URL paths.

**JSON Extraction:**
```yaml
extraction_body:
  type: extraction
  source: prefilter_stage
  fields:
    - source: HTTPReq_Body
      path: username
      alias: username
    - source: HTTPReq_Body
      path: nested.field.value
      alias: nested_value
```

**Query Parameter Extraction:**
```yaml
extraction_query:
  type: extraction
  source: prefilter_stage
  fields:
    - source: HTTPReq_Query
      path: coupon_code
      alias: coupon_code
```

**URL Path Extraction (1-based indexing):**
```yaml
# URL: /api/v1/users/123/profile
extraction_path:
  type: extraction
  source: prefilter_stage
  fields:
    - source: HTTPReq_Uri
      path: 4              # Extracts "123"
      alias: user_id
```

### 3. Normalize Stage
Merges multiple fields into one.

```yaml
normalize_user:
  type: normalize
  source: extraction_stage
  fields:
    - name: user_identifier
      sources:
        - username         # Priority 1
        - email            # Priority 2
        - user_id          # Priority 3
      default: "UNKNOWN"
```

### 4. Post-filter Stage
Filters on extracted/normalized fields.

```yaml
filter_valid:
  type: filter
  source: extraction_stage
  filters:
    - field: extracted_field
      operator: Exists
      value: True
    - field: HTTPResp_ResponseCode
      operator: GTE
      value: 200
    - field: HTTPResp_ResponseCode
      operator: LTE
      value: 299
```

### 5. Detection Stage
Applies aggregation and threshold checking.

```yaml
detection_abuse:
  type: detection
  source: filter_stage
  group_by: _ip              # Single field or array: [_ip, user_id]
  metric: distinct_count     # count, distinct_count, sum, avg, max, min
  field: target_field        # Required for distinct_count
  threshold: 5
  operator: GT               # GT, GTE, LT, LTE, EQ, NEQ
```

**Multiple Detections (different fidelity levels):**
```yaml
# Low fidelity - volume based
detection_volume:
  type: detection
  source: filter_stage
  group_by: _ip
  metric: count
  threshold: 100
  operator: GT

# High fidelity - pattern based
detection_pattern:
  type: detection
  source: filter_stage
  group_by: _ip
  metric: distinct_count
  field: otp_code
  threshold: 5
  operator: GT
```

### 6. Evidence Stage
Specifies data to collect for investigation.

```yaml
evidence_data:
  type: evidence
  source: detection_stage
  limit: 20
  fields:
    - HTTPReq_Body
    - HTTPReq_Uri
    - _ip
    - extracted_field
```

### 7. Alert Stage
Configures alert generation.

```yaml
alerts_abuse:
  type: alert                    # Type is "alert"
  source: detection_stage
  category: "Automated Threat"
  subcategory: "Threat Type"
  subject: groupby_field
  template: true                 # Required for template variables
  message: "Detected from {groupby_field} with {column:field}"
  severity: critical             # critical, high, medium, low
  title: "Alert Title"
```

---

## Field Extraction

### Supported Sources

| Source | Description | Extraction Type |
|--------|-------------|-----------------|
| `HTTPReq_Body` | Request body JSON | JSON path |
| `HTTPRes_Body` | Response body JSON | JSON path |
| `HTTPReq_Query` | Query parameters | Parameter name |
| `HTTPReq_Uri` | URL path | Numeric index (1-based) |
| `MsgHeader` | Message headers | JSON path |

### Built-in Fields

| Field | Description |
|-------|-------------|
| `_ip` | Client IP address |
| `session_id` | Session identifier |
| `APIID` | API endpoint identifier |
| `HTTPResp_ResponseCode` | HTTP response code |

---

## Filter Operators

### Comparison
| Operator | Description |
|----------|-------------|
| `EQ` | Equal to |
| `NEQ` | Not equal to |
| `GT` | Greater than |
| `GTE` | Greater than or equal |
| `LT` | Less than |
| `LTE` | Less than or equal |

### Set Operations
| Operator | Description |
|----------|-------------|
| `IN` | Value in list |
| `NOT_IN` | Value not in list |

### Pattern Matching
| Operator | Description |
|----------|-------------|
| `LIKE` | SQL pattern match (% wildcard) |
| `NOT_LIKE` | Inverse pattern match |

### Existence
| Operator | Description |
|----------|-------------|
| `Exists` | Field has non-empty value |
| `NotExists` | Field is empty or null |

---

## Detection Configuration

### Metrics

| Metric | Description | Use Case |
|--------|-------------|----------|
| `count` | Total events | Volume detection (low fidelity) |
| `distinct_count` | Unique values | Pattern detection (high fidelity) |
| `sum` | Sum numeric values | Aggregate amounts |
| `avg` | Average | Average metrics |
| `max` | Maximum | Peak values |
| `min` | Minimum | Minimum values |

### Group By
- Single field: `group_by: _ip`
- Multiple fields: `group_by: [_ip, user_id]`

---

## Alert Templates

### Template Variables

**Detection Variables:**
- `{groupby_field}` - Grouped field value
- `{detection_count}` - Event count
- `{first_seen}` - First event timestamp
- `{last_seen}` - Last event timestamp
- `{detection_timestamp}` - Detection time

**Metadata Variables:**
- `{detection_name}` - Detection stage name
- `{rule_name}` - Rule name
- `{application}` - Tenant name
- `{severity}` - Alert severity
- `{evidence_count}` - Evidence record count

**Evidence Variables:**
- `{status_2xx}` - Count of 2xx responses
- `{status_4xx}` - Count of 4xx responses
- `{column:field_name}` - Evidence column values
- `{column:field_name_count}` - Unique value count

### Template Examples
```yaml
# Basic
message: "Attack from {groupby_field} with {detection_count} attempts"

# With evidence
message: "Brute force from {groupby_field}: {status_4xx} failed, {status_2xx} successful"

# With column data
message: "Targeting {column:username_count} accounts: {column:username}"
```

---

## Common Patterns

### Pattern 1: Credential Stuffing Detection
```yaml
prefilter_login:
  type: pre_filter
  source: raw_data
  filters:
    - field: APIID
      operator: IN
      value: '@tags:account sign-in'

extraction_creds:
  type: extraction
  source: prefilter_login
  fields:
    - source: HTTPReq_Body
      path: username
      alias: username

filter_exists:
  type: filter
  source: extraction_creds
  filters:
    - field: username
      operator: Exists
      value: True

detection_stuffing:
  type: detection
  source: filter_exists
  group_by: _ip
  metric: distinct_count
  field: username
  threshold: 10
  operator: GT
```

### Pattern 2: Coupon Abuse Detection
```yaml
prefilter_coupon:
  type: pre_filter
  source: raw_data
  filters:
    - field: APIID
      operator: IN
      value: '@tags:coupon'

extraction_coupon:
  type: extraction
  source: prefilter_coupon
  fields:
    - source: HTTPReq_Body
      path: couponCode
      alias: coupon_code

detection_abuse:
  type: detection
  source: extraction_coupon
  group_by: session_id
  metric: distinct_count
  field: coupon_code
  threshold: 5
  operator: GT
```

### Pattern 3: IDOR Detection (Path Parameter)
```yaml
# URL: /api/users/{user_id}/profile
extraction_userid:
  type: extraction
  source: prefilter_stage
  fields:
    - source: HTTPReq_Uri
      path: 3
      alias: target_user_id

detection_idor:
  type: detection
  source: extraction_userid
  group_by: _ip
  metric: distinct_count
  field: target_user_id
  threshold: 10
  operator: GT
```

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| No detections | Filters too restrictive or threshold too high | Review filters, lower threshold |
| Extraction empty | Wrong path or index | Verify JSON structure, use 1-based URL index |
| Raw template in alert | Missing `template: true` | Add `template: true` to alert |
| Stage reference error | Typo in stage name | Verify stage names match exactly |
| Validation fails | Invalid YAML or missing fields | Check syntax, verify required fields |
| Tag not found | Tag doesn't exist | Run `api tags list` to verify |

### Validation Checklist
- [ ] `engine: atdv2` is set
- [ ] Stage names follow prefix patterns
- [ ] All `source` references point to existing stages
- [ ] Alert uses `type: alert` with `alerts_` prefix
- [ ] `template: true` when using template variables
- [ ] Detection has `metric`, `threshold`, `operator`
- [ ] Evidence has `fields` array
