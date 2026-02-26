---
title: Examples
contributors:
  - https://github.com/adobe
---

# Examples

This section provides complete, real-world examples demonstrating the Adobe External Actions API for all supported entity types.

## Supported Entity Types

| Entity Type | Use Case | Key Fields |
| --- | --- | --- |
| `lead` | Lead/person enrichment, scoring | `leadId`, `leadData` |
| `account` | Account enrichment, firmographics | `accountId`, `accountData` |
| `accountPerson` | Buying groups, relationship analysis | `accountId`, `accountData`, `accountPersonRelationships` |

## Service Definition Examples

### Account Enrichment Service

Service definition for B2B account enrichment with firmographic data, including i18n and custom headers:

```json
{
  "apiName": "account_enrichment_service",
  "i18n": {
    "en_US": {
      "name": "Account Enrichment Service",
      "description": "Enriches B2B account data with firmographic intelligence, technographic insights, and buying signals"
    }
  },
  "description": "Enriches account profiles with firmographic and technographic data",
  "primaryAttribute": "enrichmentType",
  "supportedEntityType": "account",
  "enableSplitPaths": true,
  "timeout": 120,
  "invocationPayloadDef": {
    "globalAttributes": [
      {
        "apiName": "apiKey",
        "i18n": {
          "en_US": {
            "displayName": "API Key",
            "description": "Your service API key for authentication"
          }
        },
        "dataType": "string",
        "required": true
      },
      {
        "apiName": "dataProvider",
        "i18n": {
          "en_US": {
            "displayName": "Data Provider",
            "description": "External data provider to use"
          }
        },
        "dataType": "string",
        "required": false
      }
    ],
    "flowAttributes": [
      {
        "apiName": "enrichmentType",
        "i18n": {
          "en_US": {
            "displayName": "Enrichment Type",
            "description": "Type of enrichment to perform (firmographic, technographic, intent)"
          }
        },
        "dataType": "string",
        "required": true
      },
      {
        "apiName": "priority",
        "i18n": {
          "en_US": {
            "displayName": "Priority",
            "description": "Processing priority level"
          }
        },
        "dataType": "string",
        "required": false
      }
    ],
    "accountFields": [
      { "serviceAttribute": "accountId", "dataType": "string", "required": true, "description": "Account ID" },
      { "serviceAttribute": "accountName", "dataType": "string", "required": true, "description": "Account name" },
      { "serviceAttribute": "website", "dataType": "string", "required": false, "description": "Company website" },
      { "serviceAttribute": "industry", "dataType": "string", "required": false, "description": "Industry" }
    ],
    "headers": [
      {
        "headerName": "X-Tenant-ID",
        "description": "Tenant identifier for multi-tenant environments"
      }
    ],
    "journeyContext": true,
    "subscriptionContext": true,
    "accessorsMetadata": [
      {
        "accessorName": "accountTier",
        "dataType": "string",
        "description": "Account tier (enterprise, mid-market, smb)"
      },
      {
        "accessorName": "enrichmentScore",
        "dataType": "integer",
        "description": "Data quality score (0-100)"
      }
    ]
  },
  "callbackPayloadDef": {
    "accountFields": [
      { "serviceAttribute": "annualRevenue", "dataType": "integer", "description": "Annual revenue" },
      { "serviceAttribute": "employeeCount", "dataType": "integer", "description": "Employee count" },
      { "serviceAttribute": "sicCode", "dataType": "string", "description": "SIC code" },
      { "serviceAttribute": "technologies", "dataType": "string", "description": "Technologies used" }
    ],
    "attributes": [
      { "serviceAttribute": "enrichmentStatus", "dataType": "string", "description": "Enrichment status" },
      { "serviceAttribute": "lastEnriched", "dataType": "datetime", "description": "Last enrichment timestamp" }
    ]
  }
}
```

### AccountPerson Buying Group Service

Service definition for buying group role identification with complete i18n:

```json
{
  "apiName": "buying_group_role_service",
  "i18n": {
    "en_US": {
      "name": "Buying Group Role Service",
      "description": "Identifies key stakeholders and their roles in B2B buying committees"
    }
  },
  "description": "Analyzes account-person relationships to identify buying committee roles",
  "primaryAttribute": "analysisType",
  "supportedEntityType": "accountPerson",
  "enableSplitPaths": true,
  "timeout": 180,
  "invocationPayloadDef": {
    "globalAttributes": [
      {
        "apiName": "apiKey",
        "i18n": {
          "en_US": {
            "displayName": "API Key",
            "description": "Service API key"
          }
        },
        "dataType": "string",
        "required": true
      },
      {
        "apiName": "buyingGroupModel",
        "i18n": {
          "en_US": {
            "displayName": "Buying Group Model",
            "description": "Buying group analysis model to use"
          }
        },
        "dataType": "string",
        "required": false
      }
    ],
    "flowAttributes": [
      {
        "apiName": "analysisType",
        "i18n": {
          "en_US": {
            "displayName": "Analysis Type",
            "description": "Analysis type (role_identification, influence_mapping, committee_complete)"
          }
        },
        "dataType": "string",
        "required": true
      },
      {
        "apiName": "minInfluenceLevel",
        "i18n": {
          "en_US": {
            "displayName": "Minimum Influence Level",
            "description": "Minimum influence threshold"
          }
        },
        "dataType": "integer",
        "required": false
      }
    ],
    "accountFields": [
      { "serviceAttribute": "accountId", "dataType": "string", "required": true, "description": "Account ID" },
      { "serviceAttribute": "accountName", "dataType": "string", "required": true, "description": "Account name" },
      { "serviceAttribute": "industry", "dataType": "string", "required": false, "description": "Industry" }
    ],
    "fields": [
      { "serviceAttribute": "email", "dataType": "string", "required": true, "description": "Email address" },
      { "serviceAttribute": "firstName", "dataType": "string", "required": false, "description": "First name" },
      { "serviceAttribute": "lastName", "dataType": "string", "required": false, "description": "Last name" },
      { "serviceAttribute": "title", "dataType": "string", "required": false, "description": "Job title" },
      { "serviceAttribute": "department", "dataType": "string", "required": false, "description": "Department" }
    ],
    "headers": [
      {
        "headerName": "X-Analysis-Version",
        "description": "Version of the analysis algorithm to use"
      }
    ],
    "journeyContext": true,
    "subscriptionContext": true,
    "accessorsMetadata": [
      {
        "accessorName": "buyingRole",
        "dataType": "string",
        "description": "Buying group role (economic_buyer, champion, influencer)"
      },
      {
        "accessorName": "influenceScore",
        "dataType": "integer",
        "description": "Influence score (0-100)"
      },
      {
        "accessorName": "engagementLevel",
        "dataType": "string",
        "description": "Engagement level (high, medium, low)"
      },
      {
        "accessorName": "committeeComplete",
        "dataType": "boolean",
        "description": "Whether all key buying roles are identified"
      }
    ]
  },
  "callbackPayloadDef": {
    "accountFields": [
      { "serviceAttribute": "buyingGroupSize", "dataType": "integer", "description": "Number of stakeholders" },
      { "serviceAttribute": "decisionTimeframe", "dataType": "string", "description": "Expected decision timeframe" }
    ],
    "fields": [
      { "serviceAttribute": "validatedTitle", "dataType": "string", "description": "Validated job title" }
    ],
    "attributes": [
      { "serviceAttribute": "analysisConfidence", "dataType": "integer", "description": "Confidence level (0-100)" },
      { "serviceAttribute": "lastAnalyzed", "dataType": "datetime", "description": "Last analysis timestamp" }
    ]
  }
}
```

**Key difference for `accountPerson`**: Requires both `accountFields` AND `fields` since it operates on leads within account context.

## Execution Request Examples

### Lead Entity Request

Request for lead/person enrichment:

```json
{
  "token": "abc123xyz789-lead-enrichment",
  "callbackUrl": "https://adobe.com/external-actions/callback/abc123xyz789",
  "context": {
    "subscription": {
      "prefix": "acme-prod",
      "imsOrgId": "12345ABCDE@AdobeOrg",
      "munchkinId": "123-ABC-456"
    },
    "journey": {
      "id": "journey-q4-2025-lead-enrichment",
      "name": "Q4 2025 Lead Enrichment Campaign",
      "status": "Live"
    },
    "admin": {
      "apiKey": "sk_live_abc123def456ghi789",
      "region": "US"
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
        "title": "VP of Sales"
      }
    },
    "flowStepContext": {
      "enrichmentType": "professional",
      "priority": "high"
    }
  },
  "actionConfig": {
    "timeout": 60,
    "pathConfig": [
      {
        "pathId": "highQuality",
        "pathDefinition": [
          { "accessor": "enrichmentScore", "operator": "greaterThanOrEqual", "values": ["80"] }
        ]
      },
      { "pathId": "default", "pathDefinition": [] }
    ]
  },
  "enableSplitPaths": true
}
```

### Account Entity Request

Request for account enrichment:

```json
{
  "token": "def456abc789-account-enrichment",
  "callbackUrl": "https://adobe.com/external-actions/callback/def456abc789",
  "context": {
    "subscription": {
      "prefix": "acme-prod",
      "imsOrgId": "12345ABCDE@AdobeOrg",
      "munchkinId": "123-ABC-456"
    },
    "journey": {
      "id": "JN-2024-001",
      "name": "Enterprise Account Nurture Journey",
      "status": "Live"
    },
    "admin": {
      "apiKey": "ak_live_1234567890abcdefghij",
      "dataProvider": "ZoomInfo"
    }
  },
  "objectData": {
    "objectType": "account",
    "objectContext": {
      "accountId": "ACC-98765",
      "accountData": {
        "accountName": "Acme Corporation",
        "website": "https://www.acme-corp.example.com",
        "industry": "Manufacturing",
        "billingCountry": "United States",
        "annualRevenue": 50000000,
        "employeeCount": 250
      }
    },
    "flowStepContext": {
      "enrichmentType": "technographic",
      "priority": "high"
    }
  },
  "actionConfig": {
    "timeout": 120,
    "pathConfig": [
      {
        "pathId": "enterprise",
        "pathDefinition": [
          { "accessor": "accountTier", "operator": "is", "values": ["enterprise"] }
        ]
      },
      { "pathId": "default", "pathDefinition": [] }
    ]
  },
  "enableSplitPaths": true
}
```

### AccountPerson Entity Request

Request for buying group analysis (leads within account context):

```json
{
  "token": "ghi789xyz456-buying-group",
  "callbackUrl": "https://adobe.com/external-actions/callback/ghi789xyz456",
  "context": {
    "subscription": {
      "prefix": "app-ent",
      "imsOrgId": "ENT9876543210@AdobeOrg",
      "munchkinId": "789-GHI-012"
    },
    "journey": {
      "id": "JN-2024-103",
      "name": "Strategic Account Buying Group Journey",
      "status": "Live"
    },
    "admin": {
      "apiKey": "ak_enterprise_abc123def456",
      "buyingGroupModel": "enterprise-b2b"
    }
  },
  "objectData": {
    "objectType": "accountPerson",
    "objectContext": {
      "accountId": "ACC-54321",
      "accountData": {
        "accountName": "Global Manufacturing Solutions Inc",
        "industry": "Industrial Manufacturing",
        "annualRevenue": 250000000,
        "employeeCount": 1500
      },
      "accountPersonRelationships": [
        {
          "accountPersonRelId": 9001,
          "accountId": 54321,
          "personId": 12345,
          "leadData": {
            "email": "michael.chen@globalmfg.example.com",
            "firstName": "Michael",
            "lastName": "Chen",
            "title": "Chief Technology Officer",
            "department": "IT"
          }
        },
        {
          "accountPersonRelId": 9002,
          "accountId": 54321,
          "personId": 12346,
          "leadData": {
            "email": "jennifer.wong@globalmfg.example.com",
            "firstName": "Jennifer",
            "lastName": "Wong",
            "title": "VP of Operations",
            "department": "Operations"
          }
        }
      ]
    },
    "flowStepContext": {
      "analysisType": "committee_complete",
      "minInfluenceLevel": 70
    }
  },
  "actionConfig": {
    "timeout": 180,
    "pathConfig": [
      {
        "pathId": "complete_committee",
        "pathDefinition": [
          { "accessor": "committeeComplete", "operator": "is", "values": ["true"] }
        ]
      },
      {
        "pathId": "high_engagement",
        "pathDefinition": [
          { "accessor": "engagementLevel", "operator": "is", "values": ["high"] }
        ]
      },
      { "pathId": "default", "pathDefinition": [] }
    ]
  },
  "enableSplitPaths": true
}
```

## Callback Response Examples

### Success Response

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
        "directPhone": "+1-555-0100 ext. 1234",
        "linkedInUrl": "https://linkedin.com/in/johndoe",
        "title": "VP of Sales",
        "seniority": "Vice President",
        "department": "Sales",
        "accessorValues": {
          "enrichmentScore": 92,
          "dataQuality": "high"
        }
      }
    }
  ]
}
```

### Error Response

```json
{
  "objectData": [
    {
      "activityData": {
        "success": false,
        "errorCode": "ENRICHMENT_FAILED",
        "reason": "External enrichment API returned 503 Service Unavailable"
      },
      "leadData": {
        "id": 67890,
        "email": "jane.smith@example.com"
      }
    }
  ]
}
```

## Testing with `cURL`

### Submit Execution Request

```bash
# Test your service's execution endpoint
curl -X POST https://your-service.com/submitAsyncAction \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d @examples/execution-requests/lead-request.json
```

### Send Callback Response

```bash
# Test callback to Adobe (in development, use a test endpoint)
curl -X POST https://adobe-test.com/callback/YOUR_CORRELATION_ID \
  -H "Content-Type: application/json" \
  -H "X-Api-Key: YOUR_API_KEY" \
  -H "X-Request-Id: YOUR_CORRELATION_ID" \
  -d @examples/callback-responses/lead-success.json
```

## Testing with Postman

For a more interactive testing experience, use our [Postman Collection](/docs/postman).

The Postman collection includes:

- Pre-configured requests for all endpoints
- Environment variable templates
- Request/response validation
- Mock server support

## Related Documentation

- [Data Flow Overview](/docs/data-flow) - Understand the complete data flow
- [Service Definition](/docs/service-definition) - Detailed service definition guide
- [Execution Request](/docs/execution-request) - Execution request structure
- [Callback Response](/docs/callback-response) - Callback response format
- [Postman Collection](/docs/postman) - Interactive API testing

