---
title: Data Flow
contributors:
  - https://github.com/adobe
---

# External Actions Data Flow

This document provides a comprehensive overview of the data flow for the Adobe External Actions API.

## Overview

The Adobe External Actions API enables external services to integrate with Adobe Journey Optimizer B2B Edition (AJOB2B) through custom journey actions.

The integration flow consists of:

1. Service Definition: External service declares capabilities via `/getServiceDefinition`
1. Execution Request: Adobe sends entity data to service via `/submitAsyncAction`
1. Async Processing: Service processes data
1. Callback Response: Service returns results to Adobe

## Prerequisites

Before integrating, your service must provide an OpenAPI 3.0.x compliant specification that includes:

- OpenAPI 3.0.x version
- Three required endpoints: `/getServiceDefinition`, `/submitAsyncAction`, `/status`
- At least one security scheme: `apiKey`, oauth2, or `basicAuth`

👉 **See [OpenAPI Spec Requirements](/docs/openapi-spec-requirements)** for complete details on creating your specification.

## Service Definition Flow

### Prerequisites: Create OpenAPI Specification

The first step is creating your OpenAPI 3.0.x specification with the required endpoints and security schemes. See [OpenAPI Spec Requirements](/docs/openapi-spec-requirements).

### Service Provider Exposes `/getServiceDefinition` Endpoint

The service provider must expose a service definition endpoint that declares:

**Key Properties:**

- `apiName`: Unique identifier for the service
- `supportedEntityType`: Single entity type supported (lead, account, or `accountPerson`)
- `enableSplitPaths`: Boolean indicating split path decisioning support
- `invocationPayloadDef`: Defines what data the service needs
- `callbackPayloadDef`: Defines what data the service can update

**Conditional Requirements** (enforced at ServiceDefinition level):

- If `supportedEntityType: lead` → requires `fields` in both invocation and callback payload defs
- If `supportedEntityType: account` → requires `accountFields` in both invocation and callback payload defs
- If `supportedEntityType: accountPerson` → requires `accountPersonRelationships` in the invocation payload; optionally supports `fields` and/or `accountFields` in the callback payload for attribute updates

See the [Service Definition Guide](/docs/service-definition) for details.

## Payload Definition Structure

### InvocationPayloadDef

Defines the structure of data that is sent from Adobe to the external service.

**Key Properties:**

- `globalAttributes`: Configuration attributes set in Admin UI
- `flowAttributes`: Flow-step-specific attributes set in Journey UI
- `fields`: Lead/person field mappings
- `accountFields`: Account field mappings
- `headers`: Custom headers for API calls
- `journeyContext`: Boolean - include journey metadata
- `subscriptionContext`: Boolean - include subscription metadata
- `accessorsMetadata`: Path condition accessor metadata (required when `enableSplitPaths: true`)

### CallbackPayloadDef

Defines the structure of data that can be sent BACK from the external service.

**Key Properties:**

- `fields`: Lead/person fields that can be updated (optional - only if supporting person attribute updates)
- `accountFields`: Account fields that can be updated (optional - only if supporting account attribute updates)

## Execution Request Flow

See [Execution Request Documentation](/docs/execution-request) for complete details on the request structure, including:

- Request structure and required fields
- Entity-specific data formats (lead, account, `accountPerson`)
- Context data (subscription, journey, admin)
- Action configuration
- Custom headers

## Callback Response Flow

See [Callback Response Documentation](/docs/callback-response) for complete details on the callback structure, including:

- Required callback headers
- Response structure by entity type
- Token overrides for journey routing
- Error handling in callbacks

## Field Mapping Lifecycle

### Service Definition Phase

The service provider declares fields in the service definition:

```yaml
callbackPayloadDef:
  fields:
    - serviceAttribute: "email"
      dataType: "email"
      required: false
      description: "Person email address"
```

### Admin Configuration Phase

The admin maps service fields to Adobe fields:
- Service field `email` → Adobe field `Email Address`

### Execution Phase

Adobe sends mapped data to the service:
```json
{
  "leadData": {
    "email": "john@example.com"
  }
}
```

### Callback Phase

The service returns updated data:
```json
{
  "leadData": {
    "id": 12345,
    "email": "updated@example.com"
  }
}
```

### Update Phase

Adobe updates records with the returned data.

## Path Condition Accessors

Path condition accessors enable external services to influence journey routing decisions. See [Path Condition Accessors](/docs/path-condition-accessors/) for details.

**Key Points:**

- Accessor values are embedded directly in entity data (`accessorValues` property)
- Each entity can have its own accessor values
- Accessors can be used in journey path conditions for dynamic routing

## Error Handling

See [Error Handling Documentation](/docs/error-handling) for details on:

- Error response structure
- Standard error codes
- Error handling best practices

## Complete End-to-End Example

### Service Definition

```yaml
supportedEntityType: lead
enableSplitPaths: true
invocationPayloadDef:
  fields:
    - serviceAttribute: email
      dataType: email
    - serviceAttribute: company
      dataType: string
  accessorsMetadata:
    - accessorName: enrichmentScore
      dataType: integer
callbackPayloadDef:
  fields:
    - serviceAttribute: email
      dataType: email
    - serviceAttribute: enrichedData
      dataType: string
```

### Execution Request

```json
{
  "token": "abc123",
  "callbackUrl": "https://adobe.com/callback",
  "objectData": {
    "objectType": "lead",
    "objectContext": {
      "leadId": "12345",
      "leadData": {
        "email": "john@example.com",
        "company": "Acme Corp"
      }
    }
  },
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
      }
    ]
  }
}
```

### Callback Response

```json
{
  "token": "abc123",
  "objectData": [
    {
      "activityData": {
        "success": true
      },
      "leadData": {
        "id": 12345,
        "email": "john@example.com",
        "enrichedData": "Premium customer - Tech industry",
        "accessorValues": {
          "enrichmentScore": 92
        }
      }
    }
  ]
}
```

### Journey Routing

Based on `accessorValues.enrichmentScore = 92`:
- Condition: `my.enrichmentScore >= 80` evaluates to `true`
- Result: Lead takes "`highValue`" path

## Additional Resources

- [OpenAPI Specification](../provider-canonical-openapi.yaml)
- [Service Definition Guide](/docs/service-definition)
- [Execution Request](/docs/execution-request)
- [Callback Response](/docs/callback-response)
- [Examples](/docs/examples)

