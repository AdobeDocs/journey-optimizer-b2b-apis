---
title: Path Condition Accessors
---

# Path Condition Accessors

## Overview

Path condition accessors allow external services to provide computed values that can be used in journey path conditions for dynamic routing decisions. This enables services to influence the journey path based on their processing logic.

## When to Use Path Condition Accessors

Use path condition accessors when you want to:

- Implement dynamic split path decisioning based on external service logic
- Route entities differently based on enrichment results
- Apply scoring or classification that affects journey flow


## Configuration

### Enable in Service Definition

```yaml
serviceDefinition:
  enableSplitPaths: true
  invocationPayloadDef:
    accessorsMetadata:
      - accessorName: enrichmentScore
        dataType: integer
        description: Data quality score (0-100)
        constraints:
          minValue: 0
          maxValue: 100
      - accessorName: treatmentId
        dataType: string
        description: Recommended treatment
        constraints:
          allowedValues: ["premium", "standard", "basic"]
```

### Admin Creates Path Conditions

In the journey builder, admins create path conditions using the defined accessors:

**Path 1: High Value**

- Condition: `my.enrichmentScore >= 80`

**Path 2: Medium Value**

- Condition: `my.treatmentId == 'standard'`

**Path 3: Default**

- Default path (no condition)

### Service Returns Accessor Values

In the callback response, include `accessorValues`:

```json
{
  "leadData": {
    "id": 12345,
    "email": "john@example.com",
    "accessorValues": {
      "enrichmentScore": 92,
      "treatmentId": "premium"
    }
  }
}
```

### Adobe Routes Based on Accessor Values

Adobe evaluates path conditions using the accessor values and routes the entity accordingly.

## Accessor Value Structure

### Location

Accessor values are embedded directly in the entity data:

| Entity Type | Location |
| --- | --- |
| Lead | `leadData.accessorValues` |
| Account | `accountData.accessorValues` |
| AccountPerson | `accountPersonData[].accessorValues` (per relationship) |

### Format

```json
{
  "accessorValues": {
    "accessorName1": value1,
    "accessorName2": value2
  }
}
```

**Rules:**

- Keys must match `accessorName` from `accessorsMetadata`
- Values must match the defined `dataType`
- All accessors are optional (omitted accessors evaluate to null)

## Data Types

| Type | Description | Example |
| --- | --- | --- |
| `integer` | Whole numbers | 42 |
| `float` | Decimal numbers | 3.14 |
| `string` | Text values | "premium" |
| `boolean` | True/false | true |

## Accessor Constraints

Define constraints to help admins create valid conditions:

### Numeric Constraints

```yaml
accessorsMetadata:
  - accessorName: score
    dataType: integer
    constraints:
      minValue: 0
      maxValue: 100
```

Usage in conditions:
- `my.score >= 80`
- `my.score < 50`

### String Constraints

```yaml
accessorsMetadata:
  - accessorName: tier
    dataType: string
    constraints:
      allowedValues: ["platinum", "gold", "silver", "bronze"]
```

Usage in conditions:

- `my.tier == 'platinum'`
- `my.tier IN ['platinum', 'gold']`

### Boolean Constraints

```yaml
accessorsMetadata:
  - accessorName: isQualified
    dataType: boolean
```

Usage in conditions:

- `my.isQualified == true`
- `my.isQualified`

## Examples by Entity Type

### Lead Path Condition Accessors

**Service Definition:**

```yaml
enableSplitPaths: true
invocationPayloadDef:
  accessorsMetadata:
    - accessorName: enrichmentScore
      dataType: integer
    - accessorName: dataQuality
      dataType: string
      constraints:
        allowedValues: ["high", "medium", "low"]
    - accessorName: isVerified
      dataType: boolean
```

**Callback Response:**

```json
{
  "leadData": {
    "id": 12345,
    "email": "john@example.com",
    "accessorValues": {
      "enrichmentScore": 92,
      "dataQuality": "high",
      "isVerified": true
    }
  }
}
```

**Journey Paths:**

- Premium Path: `my.enrichmentScore >= 80`
- Standard Path: `my.dataQuality == 'high'`
- Default Path: All others

### Account Path Condition Accessors

**Service Definition:**

```yaml
enableSplitPaths: true
invocationPayloadDef:
  accessorsMetadata:
    - accessorName: accountScore
      dataType: integer
    - accessorName: tier
      dataType: string
      constraints:
        allowedValues: ["enterprise", "business", "startup"]
    - accessorName: fitScore
      dataType: float
```

**Callback Response:**
```json
{
  "accountData": {
    "id": 67890,
    "accountName": "Acme Corp",
    "accessorValues": {
      "accountScore": 95,
      "tier": "enterprise",
      "fitScore": 8.7
    }
  }
}
```

**Journey Paths:**

- Enterprise Path: `my.tier == 'enterprise'`
- High Potential Path: `my.accountScore >= 80`
- Default Path: All others

### AccountPerson Path Condition Accessors

**Key Feature**: Each person-account relationship can have its own accessor values for relationship-specific routing.

**Service Definition:**

```yaml
enableSplitPaths: true
invocationPayloadDef:
  accessorsMetadata:
    - accessorName: engagementScore
      dataType: integer
    - accessorName: persona
      dataType: string
      constraints:
        allowedValues: ["decision_maker", "influencer", "user", "blocker"]
    - accessorName: priority
      dataType: string
      constraints:
        allowedValues: ["critical", "high", "medium", "low"]
```

**Callback Response:**

```json
{
  "accountPersonData": [
    {
      "accountPersonRelId": 111,
      "accountId": 67890,
      "personId": 12345,
      "email": "john@acme.com",
      "title": "VP Sales",
      "accessorValues": {
        "engagementScore": 92,
        "persona": "decision_maker",
        "priority": "critical"
      }
    },
    {
      "accountPersonRelId": 112,
      "accountId": 67890,
      "personId": 12346,
      "email": "jane@acme.com",
      "title": "Manager",
      "accessorValues": {
        "engagementScore": 65,
        "persona": "influencer",
        "priority": "medium"
      }
    }
  ]
}
```

**Journey Paths:**

- VIP Path: `my.persona == 'decision_maker'`
- High Engagement Path: `my.engagementScore >= 80`
- Nurture Path: `my.engagementScore < 60`

**Routing Result**:

- John (decision_maker, critical priority) → VIP Path
- Jane (influencer, medium priority) → Nurture Path

## Path Configuration in Execution Request

Adobe sends path configuration in the execution request:

```json
{
  "actionConfig": {
    "pathConfig": [
      {
        "pathId": "highValue",
        "pathDefinition": [
          {
            "accessor": "enrichmentScore",
            "operator": "greaterThanOrEqual",
            "values": ["80"]
          }
        ]
      },
      {
        "pathId": "default",
        "pathDefinition": []
      }
    ]
  },
  "enableSplitPaths": true
}
```

**Your Service:**

- Review `pathDefinition` to understand which accessors are needed
- Return appropriate values in `accessorValues`
- Adobe will evaluate conditions and route accordingly

## Common Use Cases

### Lead Scoring & Routing

```yaml
accessorsMetadata:
  - accessorName: leadScore
    dataType: integer
  - accessorName: segment
    dataType: string
    constraints:
      allowedValues: ["hot", "warm", "cold"]
```

**Callback:**

```json
{
  "accessorValues": {
    "leadScore": 85,
    "segment": "hot"
  }
}
```

**Routing:**

- Hot leads (score >= 80) → Sales team path
- Warm leads (50-79) → Nurture campaign
- Cold leads (< 50) → Long-term nurture


### Risk Assessment

```yaml
accessorsMetadata:
  - accessorName: riskLevel
    dataType: string
    constraints:
      allowedValues: ["low", "medium", "high"]
  - accessorName: complianceScore
    dataType: integer
```

**Callback:**
```json
{
  "accessorValues": {
    "riskLevel": "low",
    "complianceScore": 95
  }
}
```

**Routing:**

- Low risk + high compliance → Fast track
- Medium risk → Standard review
- High risk → Manual review

### Buying Group Roles

```yaml
accessorsMetadata:
  - accessorName: buyingGroupRole
    dataType: string
    constraints:
      allowedValues: ["champion", "economic_buyer", "decision_maker", "influencer", "user"]
  - accessorName: engagementLevel
    dataType: string
    constraints:
      allowedValues: ["high", "medium", "low"]
```

**Callback (AccountPerson):**

```json
{
  "accountPersonData": [
    {
      "accountPersonRelId": 111,
      "accessorValues": {
        "buyingGroupRole": "decision_maker",
        "engagementLevel": "high"
      }
    }
  ]
}
```

**Routing:**

- Decision makers with high engagement → Executive outreach
- Influencers → Educational content
- Users → Product demos

## Best Practices

### Define Clear Accessors

- Use descriptive names: `enrichmentScore` not `score1`
- Document expected value ranges
- Provide constraints for validation

### Return Consistent Values

- Always return same data type
- Handle missing data gracefully (omit accessor or use default)
- Validate values before sending

### Design Meaningful Paths

- Create mutually exclusive conditions when possible
- Always have a default path
- Test path logic with sample accessor values

### Document Accessor Logic

- Explain how accessor values are computed
- Provide examples of accessor values and resulting paths
- Document any dependencies or prerequisites

### Handle Edge Cases

- Missing data → Omit accessor (evaluates to null)
- Invalid values → Log error and use default
- Timeout → Send partial accessors if possible

## Troubleshooting

| Issue | Cause | Solution |
| --- | --- | --- |
| Path not taken | Accessor value doesn't match condition | Verify accessor value and condition syntax |
| Accessor ignored | Accessor name doesn't match definition | Check spelling of `accessorName` |
| Type error | Accessor value type mismatch | Ensure value matches defined `dataType` |
| All entities go to default | Accessors not returned in callback | Include `accessorValues` in callback response |
| Split paths not working | `enableSplitPaths` not set | Ensure service definition has `enableSplitPaths: true` |

## Advanced Patterns

### Time-based Routing

Include temporal factors:

```yaml
accessorsMetadata:
  - accessorName: recencyScore
    dataType: integer
  - accessorName: urgency
    dataType: string
    constraints:
      allowedValues: ["immediate", "soon", "later"]
```

## Next Steps

- Review [Callback Response](/docs/callback-response/) for callback structure
- See [Examples](/docs/examples/) for complete path condition accessor examples
- Review [Service Definition](/docs/service-definition/) for accessor metadata configuration
