---

title: Callback Response
contributors:
  - https://github.com/adobe
---

# Callback Response

After processing an execution request, your service must send the results back to Adobe through the callback URL.

## Required Headers

```
X-Callback-Token: {tokenFromExecutionRequest}
X-Api-Key: {customerApiKey}
X-Request-Id: {requestId}
Authorization: Bearer {accessToken}
x-gw-ims-org-id: {imsOrgId}
Content-Type: application/json
```

### Authentication Setup

To generate the `accessToken` for the `Authorization` header, set up [OAuth Server-to-Server credentials](https://developer.adobe.com/developer-console/docs/guides/authentication/ServerToServerAuthentication/) in Adobe Developer Console for your IMS Organization and Client.

## Request Structure

```json
{
  "objectData": [
    {
      "activityData": {...},
      "leadData": {...}  // OR accountData, OR accountPersonData
    }
  ]
}
```

**Note:** `objectData` is an array to support future batch processing, but currently contains a single element.

## Activity Data

Every callback must include activity data that indicates success/failure:

```json
{
  "activityData": {
    "success": true,
    "errorCode": null,
    "reason": null
  }
}
```

### Success Response

```json
{
  "activityData": {
    "success": true
  }
}
```

### Error Response

```json
{
  "activityData": {
    "success": false,
    "errorCode": "ENRICHMENT_FAILED",
    "reason": "External API rate limit exceeded"
  }
}
```

**Best Practices**

- Use meaningful, consistent error codes.
- Provide actionable error messages.
- Log errors for troubleshooting.

## Callback Structure by Entity Type

### Lead Callback

```json
{
  "objectData": [
    {
      "activityData": {
        "success": true
      },
      "leadData": {
        "id": 12345,
        "email": "john.doe@acme.com",
        "firstName": "John",
        "lastName": "Doe",
        "mobilePhone": "+1-555-0101",
        "linkedInUrl": "https://linkedin.com/in/johndoe",
        "customField1": "enriched-value",
        "accessorValues": {
          "enrichmentScore": 92,
          "dataQuality": "high",
          "treatmentId": "premium"
        }
      }
    }
  ]
}
```

**Required Fields:**

- `id`: Lead/Person identifier (must match `leadId` from request)

**Optional Fields:**

- Any fields defined in `callbackPayloadDef.fields` - Only include if you want to update those lead attributes
- `accessorValues`: Path condition accessor values (when `enableSplitPaths: true`)

**Important:** Only include lead field data in your callback if your service wants to update those attributes.

### Account Callback

```json
{
  "objectData": [
    {
      "activityData": {
        "success": true
      },
      "accountData": {
        "id": 67890,
        "accountName": "Acme Corporation",
        "industry": "Technology",
        "revenue": 2000000,
        "employees": 5500,
        "website": "https://acme.com",
        "customAccountField1": "enriched",
        "accessorValues": {
          "accountScore": 95,
          "tier": "enterprise",
          "region": "EMEA"
        }
      }
    }
  ]
}
```

**Required Fields:**

- `id`: Account identifier (must match `accountId` from request)

**Optional Fields:**

- Any fields defined in `callbackPayloadDef.accountFields` - Only include fields that you want to update.
- `accessorValues`: Path condition accessor values (when `enableSplitPaths: true`)

**Important:** Only include account fields in your callback that your service will update.

### AccountPerson Callback

For `accountPerson` entity type, the callback structure supports:

- Account-level updates through `accountData` (applies to all relationships)
- Person/lead updates through `leadData` inside each `accountPersonData` object (per person)
- Relationship identification through the `accountPersonData` array (REQUIRED)
- Relationship-specific accessor values for split path decisioning

**Structure mirrors the execution request for consistency.**

```json
{
  "objectData": [
    {
      "activityData": {
        "success": true,
        "analysisModel": "enterprise-b2b-v2.5",
        "leadsAnalyzed": 3,
        "rolesIdentified": 3
      },
      "accountData": {
        "id": 54321,
        "buyingGroupSize": 7,
        "decisionTimeframe": "Q2 2024",
        "committeeCompleteness": 0.85,
        "accountEngagementScore": 88,
        "opportunityStage": "evaluation"
      },
      "accountPersonData": [
        {
          "accountPersonRelId": 9001,
          "accountId": 54321,
          "personId": 12345,
          "leadData": {
            "email": "john.cto@acme.com",
            "title": "Chief Technology Officer",
            "mobilePhone": "+1-555-0101"
          },
          "accessorValues": {
            "buyingRole": "economic_buyer",
            "influenceScore": 95,
            "engagementLevel": "high"
          }
        },
        {
          "accountPersonRelId": 9002,
          "accountId": 54321,
          "personId": 12346,
          "leadData": {
            "email": "jane.vp@acme.com",
            "title": "VP of Operations"
          },
          "accessorValues": {
            "buyingRole": "champion",
            "influenceScore": 88,
            "engagementLevel": "high"
          }
        }
      ]
    }
  ]
}
```

**accountData Structure (Optional)**

- Required (if present): `id` (account identifier)
- Optional: Any account fields that are defined in `callbackPayloadDef.accountFields`


**accountPersonData Array Structure (REQUIRED)**

- Required Fields (per relationship):
  - `accountPersonRelId`: Relationship identifier (must match ID from execution request)
  - `accountId`: Account identifier (must match ID from execution request)
  - `personId`: Person/Lead identifier (must match ID from execution request)
- Optional Fields:
  - `leadData`: Person/lead attribute updates (only include if your service wants to update person attributes)
    - Fields must be defined in `callbackPayloadDef.fields`
    - Only include fields you want to update
  - `accessorValues`: Relationship-specific accessor values for split path decisioning

**Supported:**

- ✅ Account attribute updates via `accountData` (applies to all relationships)
- ✅ Person/lead attribute updates via `accountPersonData[].leadData` (per person, optional)
- ✅ Relationship-specific accessor values via `accountPersonData[].accessorValues`
- ✅ Relationship identification via `accountPersonData` (REQUIRED)

**NOT Supported (This Release):**

- ❌ Relationship metadata updates (role, influence, buying committee position, etc.)
- ❌ Additional properties in `accountPersonData` beyond the defined fields

**Note:** Support for relationship metadata updates may be added in future releases.

## Accessor Values for Split Path Decisioning

When `enableSplitPaths: true`, include `accessorValues` to influence journey routing.

### Structure

```json
{
  "accessorValues": {
    "accessorName1": value1,
    "accessorName2": value2
  }
}
```

- Keys: Must match `accessorName` from `invocationPayloadDef.accessorsMetadata`
- Values: Must match data types defined in the metadata

### Example

Service definition declares:

```yaml
accessorsMetadata:
  - accessorName: enrichmentScore
    dataType: integer
  - accessorName: treatmentId
    dataType: string
```

Callback includes:

```json
{
  "accessorValues": {
    "enrichmentScore": 92,
    "treatmentId": "premium"
  }
}
```

Journey condition uses:

- `my.enrichmentScore >= 80`
- `my.treatmentId == 'premium'`

### AccountPerson Token Values

Each person-account relationship can have its own token values:

```json
{
  "accountPersonData": [
    {
      "accountPersonRelId": 111,
      "accessorValues": {
        "persona": "decision_maker",
        "priority": "high"
      }
    },
    {
      "accountPersonRelId": 112,
      "accessorValues": {
        "persona": "influencer",
        "priority": "medium"
      }
    }
  ]
}
```

This enables relationship-specific routing decisions.

## Field Mapping

Only include fields that are:

- Defined in `callbackPayloadDef`
- Mapped by administrator during configuration
- Have values to update

**Example:**

Service definition:

```yaml
callbackPayloadDef:
  fields:
    - serviceAttribute: email
    - serviceAttribute: mobilePhone
    - serviceAttribute: linkedInUrl
```

Admin mapping:

- `email` → `Email Address`
- `mobilePhone` → `Mobile Phone`
- `linkedInUrl` → `LinkedIn URL`

Your callback can update any/all of these:

```json
{
  "leadData": {
    "id": 12345,
    "email": "updated@example.com",
    "mobilePhone": "+1-555-0101"
    // linkedInUrl not included (no update)
  }
}
```

## Error Handling

### Partial Success

If some data enrichment succeeds but other parts fail:

```json
{
  "activityData": {
    "success": true
  },
  "leadData": {
    "id": 12345,
    "email": "john@example.com",
    "mobilePhone": "+1-555-0101"
    // Some enrichment fields may be missing
  }
}
```

### Complete Failure

If processing completely fails:

```json
{
  "activityData": {
    "success": false,
    "errorCode": "API_ERROR",
    "reason": "External API returned 503 Service Unavailable"
  },
  "leadData": {
    "id": 12345
    // Required ID only, no updates
  }
}
```

### Error Codes

Define meaningful error codes:

| Code | Description | Use Case |
| --- | --- | --- |
| `ENRICHMENT_FAILED` | Data enrichment failed | External API error |
| `RATE_LIMIT_EXCEEDED` | Rate limit reached | Too many requests |
| `INVALID_DATA` | Invalid input data | Validation error |
| `TIMEOUT` | Processing timeout | Took too long |
| `NOT_FOUND` | Entity not found | No matching record |

## Complete Examples

### Success - Lead Entity

```json
{
  "objectData": [
    {
      "activityData": {
        "success": true
      },
      "leadData": {
        "id": 12345,
        "email": "john.doe@acme.com",
        "firstName": "John",
        "lastName": "Doe",
        "mobilePhone": "+1-555-0101",
        "linkedInUrl": "https://linkedin.com/in/johndoe",
        "title": "VP of Sales",
        "seniority": "Vice President",
        "department": "Sales",
        "accessorValues": {
          "enrichmentScore": 92,
          "dataQuality": "high",
          "treatmentId": "premium"
        }
      }
    }
  ]
}
```

### Failure - Account Entity

```json
{
  "objectData": [
    {
      "activityData": {
        "success": false,
        "errorCode": "ENRICHMENT_FAILED",
        "reason": "No enrichment data available for this domain"
      },
      "accountData": {
        "id": 67890
      }
    }
  ]
}
```

### Success - AccountPerson Entity

```json
{
  "objectData": [
    {
      "activityData": {
        "success": true,
        "analysisModel": "buying-group-v2.5",
        "leadsAnalyzed": 2
      },
      "accountData": {
        "id": 67890,
        "buyingGroupSize": 5,
        "accountEngagementScore": 88,
        "decisionTimeframe": "Q2 2024",
        "strategicValue": "high"
      },
      "accountPersonData": [
        {
          "accountPersonRelId": 111,
          "accountId": 67890,
          "personId": 12345,
          "leadData": {
            "email": "john@acme.com",
            "title": "CTO",
            "department": "Engineering",
            "seniority": "Executive"
          },
          "accessorValues": {
            "buyingRole": "economic_buyer",
            "influenceScore": 92,
            "engagementLevel": "high"
          }
        },
        {
          "accountPersonRelId": 112,
          "accountId": 67890,
          "personId": 12346,
          "leadData": {
            "email": "jane@acme.com",
            "title": "VP Sales",
            "department": "Sales"
          },
          "accessorValues": {
            "buyingRole": "champion",
            "influenceScore": 98,
            "engagementLevel": "critical"
          }
        }
      ]
    }
  ]
}
```

## Best Practices

- Always include required IDs: Never omit entity identifiers.
- Include accessor values when split paths are enabled.
- Update only changed fields: Only include fields that you want to update, not all fields.
- Return minimal data for read-only services: When not updating attributes, only return IDs and `activityData`.
- Validate before sending: Ensure that data types match.
- Log callback attempts: Track success/failure for debugging.

## Response from Adobe

Responses from Adobe look like:

### Success

```json
{
  "status": "accepted",
  "message": "Callback processed successfully"
}
```

and a HTTP 200 status.

### Failure

```json
{
  "error": {
    "code": "INVALID_CALLBACK_TOKEN",
    "message": "Invalid callback token"
  }
}
```

and a HTTP 400/500 status.

## Retry Logic

If callback fails:

- Implement exponential backoff.
- Retry up to 3-5 times.
- Log failures for manual intervention.
- Don't retry on 4xx errors (client errors).

## Next Steps

- Review [Path Condition Accessors](/docs/path-condition-accessors/) for split path details
- See [Examples](/docs/examples/) for more samples
- Test with [Postman collection](/docs/postman/)
