---
title: Execution Request (Compact)
---

# Execution Request (Compact)

Adobe sends an execution request when an entity reaches your External Action step in a journey or campaign.

## Endpoint

Implement:

**POST** `/submitAsyncAction`

## Envelope

Minimal shape:

```json
{
  "token": "string",
  "callbackUrl": "string",
  "objectData": {
    "objectType": "lead|account|accountPerson",
    "objectContext": {}
  }
}
```

## Top-Level Fields

| Field | Required | Type | Purpose |
| --- | --- | --- | --- |
| `token` | Yes | string | Correlation/auth token; send back in callback header `X-Callback-Token` |
| `callbackUrl` | Yes | string | Adobe callback endpoint |
| `objectData` | Yes | object | Entity payload |
| `context` | No | object | Subscription, journey, admin context |
| `objectData.flowStepContext` | No | object | Flow attributes from `invocationPayloadDef.flowAttributes` |
| `actionConfig` | No | object | Timeout and split-path configuration |
| `customHeaders` | No | object | Custom headers from service definition |
| `enableSplitPaths` | No | boolean | Enables split-path accessor collection |

## Context Blocks

| Context block | Key fields | Notes |
| --- | --- | --- |
| `context.subscription` | `prefix`, `imsOrgId`, `instanceName`, `munchkinId` | Customer/instance identity |
| `context.journey` | `id`, `name`, `status`, `startDate`, `endDate` | Journey metadata |
| `context.admin` | service-defined globals | Values from `invocationPayloadDef.globalAttributes` |

## `objectData` by Entity Type

| `objectType` | Required identifiers | Data container(s) | Notes |
| --- | --- | --- | --- |
| `lead` | `leadId` | `leadData` | Lead fields are defined by `invocationPayloadDef.fields` |
| `account` | `accountId` | `accountData` | Account fields are defined by `invocationPayloadDef.accountFields` |
| `accountPerson` | `accountId`, per-relationship `accountPersonRelId`, `accountId`, `personId` | `accountData`, `accountPersonRelationships[].leadData` | Supports account + relationship/person context |

## Relationship Object (`accountPersonRelationships[]`)

| Field | Required | Purpose |
| --- | --- | --- |
| `accountPersonRelId` | Yes | Relationship identifier |
| `accountId` | Yes | Parent account identifier |
| `personId` | Yes | Person/lead identifier |
| `leadData` | No | Person attributes |
| `relationshipMetadata` | No | Relationship context |
| `mktoGuid`, `isEmployee` | Optional | Additional relationship details |

## `actionConfig`

| Field | Required | Type | Notes |
| --- | --- | --- | --- |
| `timeout` | No | integer | Max processing time (minutes) |
| `pathConfig` | No | array | Split-path definitions when `enableSplitPaths: true` |
| `pathConfig[].pathId` | Conditional | string | Unique path id |
| `pathConfig[].pathDefinition` | Conditional | array | Accessor conditions; empty array means default path |

Condition tuple in `pathDefinition[]`:

| Field | Type | Example |
| --- | --- | --- |
| `accessor` | string | `score` |
| `operator` | string | `greaterThanOrEqual` |
| `values` | array | `["80"]` |

## `customHeaders`

Send service-specific runtime headers from admin configuration.

| Example key | Example value |
| --- | --- |
| `X-Custom-Header` | `value` |
| `X-Tenant-ID` | `tenant-123` |

## Validation Checklist

- `token`, `callbackUrl`, and `objectData` exist.
- `objectData.objectType` matches one supported entity type.
- Required IDs for that entity type are present.
- `flowStepContext` fields align with `invocationPayloadDef.flowAttributes`.
- `actionConfig.pathConfig` is only used when split paths are enabled.

## Next Steps

- Review [Callback Response](/docs/callback-response/) for required callback format
- Review [Error Handling](/docs/error-handling/) for request and callback failure behavior
