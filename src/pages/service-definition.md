---
title: Service Definition Guide
description: Define service capabilities, required fields, and outputs for the getServiceDefinition endpoint.
---

# Service Definition Guide

The service definition endpoint `/getServiceDefinition` is one of three required endpoints in your OpenAPI specification. It declares your service's capabilities, data requirements, and configuration options.

## Prerequisites

Before implementing this endpoint, ensure you have:

1. Created your OpenAPI 3.0.x specification with the required components.
1. Defined at least one security scheme (`apiKey`, oauth2, or `basicAuth`)
1. Included all three required endpoints in your spec

See [OpenAPI Spec Requirements](openapi-spec-requirements.md) for details on writing your specification.

## `GET /getServiceDefinition`

Response: `ServiceDefinition` object (application/json)

| Property | Type | Description |
| --- | --- | --- |
| `apiName` | string | Unique identifier for your service |
| `i18n` | object | Internationalization strings (at minimum `en_US` with `name` and `description`) |
| `description` | string | Human-readable service description |
| `supportedEntityType` | string | Entity type: `lead`, `account`, or `accountPerson` |
| `primaryAttribute` | string | API name of the primary attribute from flow attributes that describes the main asset |
| `invocationPayloadDef` | object | Defines data needed by your service |
| `callbackPayloadDef` | object | Defines data your service can update |
| `enableSplitPaths` | boolean | Whether split path decisioning is enabled (required, set to `true` or `false`) |
| `timeout` | integer | Optional. Callback timeout in minutes (max: 240) |

Your service must support exactly ONE entity type:

- `lead` (Lead/Person)
  - Requires `fields` in payload definitions
  - Use for: lead enrichment, scoring, qualification

- `account` (Account)
  - Requires `accountFields` in payload definitions
  - Use for: account enrichment, firmographic data, ABM scoring

- `accountPerson` (Leads within Accounts)
  - Requires BOTH `fields` AND `accountFields`
  - Use for: buying group analysis, relationship scoring, multi-touch attribution

## Authentication & Security

Your OpenAPI specification must define one of the following security schemes:

<Tab orientation="horizontal" slots="heading, code" repeat="3"/>

### API Key Authentication

```yaml
components:
  securitySchemes:
    apiKey:
      type: apiKey
      in: header          # or 'query'
      name: X-API-Key     # customize header/query param name
      description: API Key authentication

security:
  - apiKey: []
```

### OAuth 2.0

```yaml
components:
  securitySchemes:
    oauth2:
      type: oauth2
      description: OAuth2 authentication
      flows:
        authorizationCode:
          authorizationUrl: https://your-service.com/oauth/authorize
          tokenUrl: https://your-service.com/oauth/token
          refreshUrl: https://your-service.com/oauth/refreshToken
          scopes: {}
        # OR clientCredentials:
        clientCredentials:
          tokenUrl: https://your-service.com/oauth/token
          scopes: {}

security:
  - oauth2: []
```

### HTTP Basic Authentication

```yaml
components:
  securitySchemes:
    basicAuth:
      type: http
      scheme: basic
      description: HTTP Basic authentication

security:
  - basicAuth: []
```

- Adobe only allows one security scheme defined in your root-level `security` array.
- The security scheme must be both defined in both `components.securitySchemes` and in the root-level `security` array.

## Invocation Payload Definition

The invocation payload defines what data your service needs from Adobe.

```yaml
invocationPayloadDef:
  globalAttributes:
    - serviceAttribute: apiKey
      dataType: string
      required: true
      description: API key for service authentication
  flowAttributes:
    - serviceAttribute: campaignId
      dataType: string
      required: false
      description: Campaign identifier
  fields:  # Required for 'lead' and 'accountPerson'
    - serviceAttribute: email
      dataType: email
      required: true
      description: Person email address
  accountFields:  # Required for 'account' and 'accountPerson'
    - serviceAttribute: accountName
      dataType: string
      required: true
      description: Account name
  headers:
    - headerName: X-Custom-Header
      defaultValue: default-value
      description: Custom header for API calls
  journeyContext: true  # Include journey metadata
  subscriptionContext: true  # Include subscription metadata
  accessorsMetadata:  # Required when enableSplitPaths: true
    - accessorName: score
      dataType: integer
      description: Computed score for routing
```

### Field definition properties

| Property | Type | Required | Description |
| --- | --- | --- | --- |
| `serviceAttribute` | string | Yes | Field name used by your service |
| `dataType` | string | Yes | Data type (see Data Types below) |
| `required` | boolean | No | Whether field is required (defaults to false) |
| `description` | string | No | Human-readable description for admins (recommended but optional) |

### Attribute definition properties

For `globalAttributes` and `flowAttributes`, each attribute must include:

| Property | Type | Required | Description |
| --- | --- | --- | --- |
| `apiName` | string | Yes | API name for the attribute |
| `i18n` | object | Yes | Internationalized display information |
| `i18n.en_US` | object | Yes | English (US) localization (required locale) |
| `i18n.en_US.displayName` | string | Yes | Display name shown to users in the UI |
| `i18n.en_US.description` | string | No | Optional description (recommended for clarity) |
| `dataType` | string | Yes | Data type: `string`, `integer`, `float`, `boolean`, `date`, `datetime` |
| `required` | boolean | No | Whether this attribute is required (defaults to false) |

## Callback Payload Definition

Defines what data your service can update in Adobe systems.

```yaml
callbackPayloadDef:
  fields:  # Optional - only include if your service updates person attributes
    - serviceAttribute: email
      dataType: email
      required: false
      description: Updated email address
    - serviceAttribute: leadScore
      dataType: integer
      required: false
      description: Computed lead score
  accountFields:  # Optional - only include if your service updates account attributes
    - serviceAttribute: revenue
      dataType: integer
      required: false
      description: Updated revenue
  attributes:
    - serviceAttribute: processingStatus
      dataType: string
      description: Processing status
```

## Split Path Configuration

When `enableSplitPaths: true`, you must define `accessorsMetadata` in `invocationPayloadDef`.

Accessors are named values that can be used in path conditions.

```yaml
accessorsMetadata:
  - accessorName: enrichmentScore
    dataType: integer
    description: Enrichment score for routing
    constraints:
      minValue: 0
      maxValue: 100
  - accessorName: treatmentId
    dataType: string
    description: Recommended treatment
    constraints:
      allowedValues: ["premium", "standard", "basic"]
```

Admins can create path conditions using these accessors:

- `my.enrichmentScore >= 80`
- `my.treatmentId == 'premium'`

Your service returns accessor values in callbacks:

```json
{
  "accessorValues": {
    "enrichmentScore": 92,
    "treatmentId": "premium"
  }
}
```

## Example

```yaml
apiName: "dataEnrichmentService"
i18n:
  en_US:
    name: "Data Enrichment Service"
    description: "Enriches lead and account data with third-party intelligence"
description: "Enriches lead and account data with third-party intelligence"
primaryAttribute: "email"
supportedEntityType: lead
enableSplitPaths: true
timeout: 120

invocationPayloadDef:
  globalAttributes:
    - apiName: apiKey
      dataType: string
      required: true
      i18n:
        en_US:
          displayName: "API Key"
          description: "Service API key"
    - apiName: region
      dataType: string
      required: false
      i18n:
        en_US:
          displayName: "Data Region"
          description: "Data region (US, EU, APAC)"
  flowAttributes:
    - apiName: enrichmentType
      dataType: string
      required: true
      i18n:
        en_US:
          displayName: "Enrichment Type"
          description: "Type of enrichment to perform"
  fields:
    - serviceAttribute: email
      dataType: email
      required: true
      description: Person email address
    - serviceAttribute: company
      dataType: string
      required: false
      description: Company name
    - serviceAttribute: title
      dataType: string
      required: false
      description: Job title
  headers:
    - headerName: X-Tenant-ID
      description: Tenant identifier
  journeyContext: true
  subscriptionContext: true
  accessorsMetadata:
    - accessorName: enrichmentScore
      dataType: integer
      description: Data quality score (0-100)
      constraints:
        minValue: 0
        maxValue: 100
    - accessorName: dataQuality
      dataType: string
      description: Data quality level
      constraints:
        allowedValues: ["high", "medium", "low"]

callbackPayloadDef:
  fields:
    - serviceAttribute: email
      dataType: email
      description: Verified email address
    - serviceAttribute: mobilePhone
      dataType: string
      description: Enriched mobile phone
    - serviceAttribute: linkedInUrl
      dataType: url
      description: LinkedIn profile URL
    - serviceAttribute: seniority
      dataType: string
      description: Job seniority level
  attributes:
    - serviceAttribute: enrichmentStatus
      dataType: string
      description: Enrichment processing status
    - serviceAttribute: enrichmentDate
      dataType: datetime
      description: When enrichment was performed
```

## Validation

Adobe validates your service definition with:

1. Schema validation: Ensures that all required fields are present.
1. Entity type validation: Checks conditional field requirements.
   - `lead` requires `fields` in at least one of the payload defs
   - `account` requires `accountFields` in at least one of the payload defs
   - `accountPerson` requires `accountPersonRelationships` in invocation payload; optionally supports `fields` and/or `accountFields` for attribute updates
1. Split path validation: When `enableSplitPaths: true`, requires `accessorsMetadata`.
1. Data type validation: Ensures that data types are valid.

## Best Practices

- Use descriptive names: Make `serviceAttribute` names clear and intuitive.
- Provide good descriptions: Help admins understand field purposes.
- Mark fields required carefully: Only require truly necessary fields.
- Use appropriate data types: Leverage specific types (email, url) for validation.
- Define clear accessors: When using split paths, provide meaningful accessor names.
- Version your service: Update `serviceVersion` when making changes.
- Document constraints: Use `constraints` to define valid value ranges.
