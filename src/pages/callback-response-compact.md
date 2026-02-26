---
title: Callback Response (Compact)
contributors:
  - https://github.com/adobe
---

# Callback Response (Compact)

After processing an execution request, your service must POST results to Adobe using the provided callback URL.

## Required Headers

| Header | Required | Value |
| --- | --- | --- |
| `X-Callback-Token` | Yes | `token` from execution request |
| `X-Api-Key` | Yes | Customer API key |
| `X-Request-Id` | Yes | Request identifier |
| `Authorization` | Yes | `Bearer {accessToken}` |
| `x-gw-ims-org-id` | Yes | IMS organization id |
| `Content-Type` | Yes | `application/json` |

To obtain `accessToken`, configure OAuth Server-to-Server credentials in Adobe Developer Console.

## Callback Envelope

```json
{
  "objectData": [
    {
      "activityData": {
        "success": true
      }
    }
  ]
}
```

`objectData` is currently a single-item array, but remains an array for future batching.

## `activityData` Requirements

| Field | Required | Type | Notes |
| --- | --- | --- | --- |
| `success` | Yes | boolean | `true` for successful processing, `false` for processing failure |
| `errorCode` | Conditional | string | Required when `success: false` |
| `reason` | Conditional | string | Required when `success: false`; keep actionable |

Failure example:

```json
{
  "activityData": {
    "success": false,
    "errorCode": "ENRICHMENT_FAILED",
    "reason": "External API returned 503"
  }
}
```

## Entity Payload Matrix

| Entity type | Required callback object | Required IDs | Optional updates |
| --- | --- | --- | --- |
| `lead` | `leadData` | `leadData.id` | Fields in `callbackPayloadDef.fields`, `accessorValues` |
| `account` | `accountData` | `accountData.id` | Fields in `callbackPayloadDef.accountFields`, `accessorValues` |
| `accountPerson` | `accountPersonData[]` (required), `accountData` (optional) | Per relationship: `accountPersonRelId`, `accountId`, `personId` | `accountPersonData[].leadData`, relationship `accessorValues`, account-level updates |

Only include attributes you want to update.

## `accountPersonData[]` Relationship Object

| Field | Required | Notes |
| --- | --- | --- |
| `accountPersonRelId` | Yes | Must match execution request |
| `accountId` | Yes | Must match execution request |
| `personId` | Yes | Must match execution request |
| `leadData` | No | Per-person updates; fields must be in `callbackPayloadDef.fields` |
| `accessorValues` | No | Per-relationship split-path values |

Not supported in this release:

- Relationship metadata updates (role/influence/etc.)
- Additional undefined properties in `accountPersonData`

## Accessor Values (Split Path)

When `enableSplitPaths: true`, return `accessorValues` keys that match `invocationPayloadDef.accessorsMetadata.accessorName`.

| Rule | Requirement |
| --- | --- |
| Key name | Must exactly match configured accessor name |
| Value type | Must match declared accessor `dataType` |
| Scope | For `accountPerson`, values can be per relationship |

## Field Mapping Rules

Include only fields that are:

- Declared in `callbackPayloadDef`
- Mapped by administrator configuration
- Intended to be updated

## Error and Retry Behavior

| Adobe callback response | Meaning | Retry |
| --- | --- | --- |
| `200` | Accepted | No |
| `4xx` | Client/config/payload issue | No, fix first |
| `5xx` | Temporary server issue | Yes, exponential backoff (3-5 max) |

Recommended error codes:

| Code | Typical use |
| --- | --- |
| `ENRICHMENT_FAILED` | Downstream enrichment failure |
| `RATE_LIMIT_EXCEEDED` | Provider throttling |
| `INVALID_DATA` | Validation failed |
| `TIMEOUT` | Processing exceeded timeout |
| `NOT_FOUND` | No matching entity |

## Minimal Complete Patterns

Success (lead):

```json
{
  "objectData": [
    {
      "activityData": { "success": true },
      "leadData": { "id": 12345 }
    }
  ]
}
```

Failure (account):

```json
{
  "objectData": [
    {
      "activityData": {
        "success": false,
        "errorCode": "ENRICHMENT_FAILED",
        "reason": "No enrichment data available"
      },
      "accountData": { "id": 67890 }
    }
  ]
}
```

## Best Practices

- Always include required IDs, including within failure callbacks.
- To avoid unintended updates, return only changed fields.
- Validate data types before sending callback payloads.
- Log callback attempts and Adobe response codes.

## Next Steps

- Review [Path Condition Accessors](/docs/path-condition-accessors/) for split-path logic
- Review [Error Handling](/docs/error-handling/) for request/callback failure handling
