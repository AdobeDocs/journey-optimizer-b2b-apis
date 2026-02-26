---
title: Execution Request
---

# Execution Request

Adobe sends the execution request to your service when an entity (person, account, or person-in-account) reaches your External Action step in a journey or campaign flow.

## Endpoint

Your service must implement:

**POST** `/submitAsyncAction`

## Request Structure

```json
{
  "token": "string",
  "callbackUrl": "string",
  "context": {
    "subscription": {...},
    "journey": {...},
    "admin": {...}
  },
  "objectData": {
    "objectType": "string",
    "objectContext": {...},
    "flowStepContext": {...}
  },
  "actionConfig": {...},
  "customHeaders": {...},
  "enableSplitPaths": boolean
}
```

## Required Fields

| Field | Type | Description |
| --- | --- | --- |
| `token` | string | Encrypted correlation token for callback authentication. Must be sent back in `X-Callback-Token` header when calling the callback URL. |
| `callbackUrl` | string | Adobe endpoint to POST callback results |
| `objectData` | object | Entity data with type discriminator |
| `objectData.objectType` | string | Entity type: `lead`, `account`, or `accountPerson` |
| `objectData.objectContext` | object | Actual entity data |

## Optional Fields

| Field | Type | Description |
| --- | --- | --- |
| `context` | object | Contains subscription, journey, and admin context data |
| `objectData.flowStepContext` | object | Flow-specific attributes defined in `invocationPayloadDef.flowAttributes` |
| `actionConfig` | object | Action-specific configuration (timeout in minutes, split paths) |
| `customHeaders` | object | Custom headers defined in service definition |
| `enableSplitPaths` | boolean | Enable path condition accessor collection (default: false) |

## Context Data

### Subscription Context

```json
{
  "subscription": {
    "prefix": "customer-prefix",
    "imsOrgId": "org-id@AdobeOrg",
    "instanceName": "prod-instance",
    "munchkinId": "123-ABC-456"
  }
}
```

**Fields:**

- `prefix`: Customer instance prefix
- `imsOrgId`: Adobe IMS Organization ID
- `instanceName`: Instance name
- `munchkinId`: Marketo Munchkin ID

### Journey Context

```json
{
  "journey": {
    "id": "journey-123",
    "name": "Q4 Campaign",
    "status": "Live",
    "startDate": "2025-10-01T00:00:00Z",
    "endDate": "2025-12-31T23:59:59Z"
  }
}
```

**Fields:**

- `id`: Journey identifier
- `name`: Journey name
- `status`: Journey status (Draft, Live, Completed)
- `startDate`: Journey start date
- `endDate`: Journey end date

### Admin Context

```json
{
  "admin": {
    "apiKey": "service-api-key",
    "tenantId": "tenant-123",
    "region": "US"
  }
}
```

Contains global configuration data defined by `invocationPayloadDef.globalAttributes` and configured by admins.

## Object Data by Entity Type

### Lead Entity

```json
{
  "objectData": {
    "objectType": "lead",
    "objectContext": {
      "leadId": "12345",
      "leadData": {
        "email": "john@example.com",
        "firstName": "John",
        "lastName": "Doe",
        "company": "Acme Corp",
        "title": "VP Sales"
      }
    },
    "flowStepContext": {
      "campaignId": "campaign-456",
      "source": "webinar"
    }
  }
}
```

**Key Points:**

- `leadId`: Adobe system identifier (required)
- `leadData`: Contains fields defined in `invocationPayloadDef.fields`
- `flowStepContext`: Contains values for `invocationPayloadDef.flowAttributes`

### Account Entity

```json
{
  "objectData": {
    "objectType": "account",
    "objectContext": {
      "accountId": "67890",
      "accountData": {
        "accountName": "Acme Corporation",
        "industry": "Technology",
        "revenue": 1000000,
        "employees": 5000,
        "website": "https://acme.com"
      }
    },
    "flowStepContext": {
      "priority": "high"
    }
  }
}
```

**Key Points:**

- `accountId`: Adobe system identifier (required)
- `accountData`: Contains fields defined in `invocationPayloadDef.accountFields`

### AccountPerson Entity

```json
{
  "objectData": {
    "objectType": "accountPerson",
    "objectContext": {
      "accountId": "67890",
      "accountData": {
        "accountName": "Acme Corporation",
        "industry": "Technology"
      },
      "accountPersonRelationships": [
        {
          "accountPersonRelId": 111,
          "accountId": 67890,
          "personId": 12345,
          "mktoGuid": "guid-123",
          "isEmployee": true,
          "leadData": {
            "email": "john@acme.com",
            "firstName": "John",
            "title": "VP Sales"
          },
          "relationshipMetadata": {
            "role": "decision-maker",
            "influence": "high"
          }
        }
      ]
    },
    "flowStepContext": {
      "accountScore": "enterprise"
    }
  }
}
```

**Key Points:**

- Combines account-level data with multiple person relationships
- Each person has their own `leadData`
- `relationshipMetadata` provides account-person relationship context
- All relationship IDs are required

## Action Configuration

```json
{
  "actionConfig": {
    "timeout": 60,
    "pathConfig": [
      {
        "pathId": "highValue",
        "pathDefinition": [
          {
            "accessor": "score",
            "operator": "greaterThanOrEqual",
            "values": ["80"]
          }
        ]
      },
      {
        "pathId": "standard",
        "pathDefinition": []
      }
    ]
  }
}
```

**Fields:**

- `timeout`: Maximum processing time (minutes)
- `pathConfig`: Split path configuration (when `enableSplitPaths: true`)
  - `pathId`: Unique identifier for the path
  - `pathDefinition`: Array of condition definitions (accessor, operator, values)
    - Empty array for default path

## Custom Headers

```json
{
  "customHeaders": {
    "X-Custom-Header": "value",
    "X-Tenant-ID": "tenant-123"
  }
}
```

Headers defined in `invocationPayloadDef.headers` with configured values.

## Enable Split Paths

```json
{
  "enableSplitPaths": true
}
```

Indicates whether split path decisioning is enabled. When `true`, your callback should include `accessorValues` for journey routing.

## Complete Example

### Request

```json
{
  "token": "abc123xyz789",
  "callbackUrl": "https://adobe.com/callback/abc123",
  "context": {
    "subscription": {
      "prefix": "acme-prod",
      "imsOrgId": "12345@AdobeOrg",
      "instanceName": "production",
      "munchkinId": "123-ABC-456"
    },
    "journey": {
      "id": "journey-q4-2025",
      "name": "Q4 2025 Enterprise Campaign",
      "status": "Live",
      "startDate": "2025-10-01T00:00:00Z",
      "endDate": "2025-12-31T23:59:59Z"
    },
    "admin": {
      "apiKey": "sk_live_abc123...",
      "region": "US",
      "dataCenter": "east"
    }
  },
  "objectData": {
    "objectType": "lead",
    "objectContext": {
      "leadId": "12345",
      "leadData": {
        "email": "john.doe@acme.com",
        "firstName": "John",
        "lastName": "Doe",
        "company": "Acme Corporation",
        "title": "VP of Sales",
        "phone": "+1-555-0100"
      }
    },
    "flowStepContext": {
      "campaignId": "campaign-456",
      "source": "webinar-registration",
      "enrichmentType": "professional"
    }
  },
  "actionConfig": {
    "timeout": 60,
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
        "pathId": "lowValue",
        "pathDefinition": [
          {
            "accessor": "enrichmentScore",
            "operator": "lessThan",
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
  "customHeaders": {
    "X-Tenant-ID": "acme-prod",
    "X-Request-Source": "AJO"
  },
  "enableSplitPaths": true
}
```

## Response

Your service should respond with HTTP 200 to acknowledge receipt:

```json
{
  "requestId": "req-12345",
  "status": "accepted",
  "message": "Request accepted for processing"
}
```

Then process asynchronously and send results to `callbackUrl`.

## Best Practices

1. Store token: Save it to include in the X-Callback-Token header when calling back
1. Respect timeout: Complete processing within configured timeout
1. Handle missing fields: Not all optional fields will be present
1. Echo secondary IDs: Must be included in callback if present
1. Process asynchronously: Don't block the response
1. Use provided context: Leverage subscription/journey data for logging/routing
1. Secure your endpoint: Implement authentication/authorization

## Error Handling

If your service cannot accept the request:

```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Missing required field: email"
  }
}
```

HTTP status codes:

- `200`: Request accepted
- `400`: Invalid request (malformed)
- `401`: Unauthorized
- `500`: Internal server error

## Next Steps

- Implement async processing logic
- See [Callback Response](/docs/callback-response/) for returning results
- Review [Examples](/docs/examples/) for more samples
