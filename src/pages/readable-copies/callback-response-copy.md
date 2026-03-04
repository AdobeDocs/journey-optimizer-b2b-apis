---
title: Callback Response
description: Detailed guide for sending callback responses to Adobe after asynchronous processing.
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

Note: `objectData` is an array to support future batch processing, but currently contains a single element.

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

### Best Practices

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

### Required Fields

| Field | Requirement | Description |
| --- | --- | --- |
| `id` | Required | Lead/Person identifier (must match `leadId` from request) |

### Optional Fields

| Field | Requirement | Description |
| --- | --- | --- |
| `callbackPayloadDef.fields` mapped values | Optional | Include only lead attributes your service wants to update |
| `accessorValues` | Optional | Path condition accessor values when `enableSplitPaths: true` |

Important: Only include lead field data in your callback if your service wants to update those attributes.

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

### Required Fields

| Field | Requirement | Description |
| --- | --- | --- |
| `id` | Required | Account identifier (must match `accountId` from request) |

### Optional Fields

| Field | Requirement | Description |
| --- | --- | --- |
| `callbackPayloadDef.accountFields` mapped values | Optional | Include only account attributes your service wants to update |
| `accessorValues` | Optional | Path condition accessor values when `enableSplitPaths: true` |

Important: Only include account fields in your callback that your service will update.

### AccountPerson Callback

For `accountPerson` entity type, the callback structure supports:

| Capability | Description |
| --- | --- |
| Account-level updates | Use `accountData` (applies to all relationships) |
| Person/lead updates | Use `leadData` inside each `accountPersonData` object (per person) |
| Relationship identification | Provided by the required `accountPersonData` array |
| Relationship-specific accessors | Use `accountPersonData[].accessorValues` for split path decisioning |

Structure mirrors the execution request for consistency.

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

### accountData Structure

| Field | Requirement | Description |
| --- | --- | --- |
| `id` | Required when `accountData` is present | Account identifier |
| `callbackPayloadDef.accountFields` mapped values | Optional | Account fields to update |


### accountPersonData Array Structure

| Field | Requirement | Description |
| --- | --- | --- |
| `accountPersonRelId` | Required (per relationship) | Relationship identifier (must match execution request) |
| `accountId` | Required (per relationship) | Account identifier (must match execution request) |
| `personId` | Required (per relationship) | Person/Lead identifier (must match execution request) |
| `leadData` | Optional | Person/lead attribute updates from `callbackPayloadDef.fields` |
| `accessorValues` | Optional | Relationship-specific accessor values for split path decisioning |

### Support Matrix

| Capability | Status | Notes |
| --- | --- | --- |
| Account attribute updates via `accountData` | Supported | Applies to all relationships |
| Person/lead updates via `accountPersonData[].leadData` | Supported | Per person, optional |
| Relationship-specific values via `accountPersonData[].accessorValues` | Supported | Used for split paths |
| Relationship identification via `accountPersonData` | Supported | Required |
| Relationship metadata updates (role, influence, buying committee position) | Not supported (this release) | May be added later |
| Additional properties in `accountPersonData` beyond defined fields | Not supported (this release) | Rejected by schema |

Note: Support for relationship metadata updates may be added in future releases.

## Split Path Accessors

When `enableSplitPaths: true`, include `accessorValues` in the entity payload (`leadData`, `accountData`, or `accountPersonData[]`) and keep keys/data types aligned with `accessorsMetadata` from the service definition.

See [Path Condition Accessors](path-condition-accessors.md) for full accessor design and routing examples.

## Field Mapping

Only send fields that are declared in `callbackPayloadDef`, mapped by the admin, and intended to be updated by this callback.

## Callback Error Model

This page focuses on callback payload structure. Keep complete error taxonomy, response handling, and retry strategy in [Error Handling](error-handling.md).

At minimum in callback failures:

| Field | Requirement |
| --- | --- |
| `activityData.success` | `false` |
| `activityData.errorCode` | Stable, machine-readable code |
| `activityData.reason` | Human-readable reason |
| Entity ID (`id`, `accountId`, or relationship IDs) | Always include required identifiers |

## Additional Examples

For broader scenario payloads and end-to-end samples, see [Examples](examples.md) and [Postman Collection](../postman.md.md).
