---
title: Postman Collection
description: Setup and usage guide for the Postman collection used to test External Actions endpoints.
hideBreadcrumbs: true
---

# Postman Collection

The Postman collection provides a complete set of pre-configured requests for testing all API endpoints, including service discovery, execution requests, and callback responses.

## Collection Contents

The collection includes requests for:

- Service Definition - Get service capabilities and configuration
- Execution Requests - Test entity submissions (lead, account, `accountPerson`)
- Callback Responses - Test callback submissions to Adobe
- Health Check - Verify service availability

## Setup

1. Download [external-actions-postman.json](../../static/external-actions-postman.json) .
1. Open Postman Desktop or Web.
1. Click Import and select `external-actions-postman.json` from your download location.
1. Click Import.

### Configure Environment Variables

Create a new environment in Postman with these variables:

| Variable | Description | Example |
| --- | --- | --- |
| `serviceUrl` | Your service base URL | `https://your-service.com` |
| `adobeCallbackUrl` | Adobe callback endpoint | `https://adobe.com/external-actions/callback` |
| `apiKey` | API key for authentication | `your-api-key` |
| `callbackToken` | Callback token from execution request | `test-token-123` |
| `bearerToken` | OAuth bearer token (if using OAuth) | `your-bearer-token` |

or create a Postman environment file:

```json
{
  "name": "External Actions - Development",
  "values": [
    {
      "key": "serviceUrl",
      "value": "https://dev.your-service.com",
      "enabled": true
    },
    {
      "key": "adobeCallbackUrl",
      "value": "https://adobe-dev.com/callback",
      "enabled": true
    },
    {
      "key": "apiKey",
      "value": "dev-api-key",
      "enabled": true
    },
    {
      "key": "callbackToken",
      "value": "",
      "enabled": true
    }
  ]
}
```

### Test Your First Request

1. Select Get Service Definition request.
1. Ensure your environment is selected.
1. Click Send.
1. Verify the response matches your service definition.

## Testing Workflows

### Test Service Definition

Verify your service declares capabilities correctly.

1. Select Get Service Definition
1. Click Send
1. Verify response contains:
   - `supportedEntityType`
   - `invocationPayloadDef`
   - `callbackPayloadDef`
   - `enableSplitPaths` (if applicable)

### Test Execution Request

Verify your service accepts and processes execution requests.

1. Select an execution request:
   - Execute - Lead Entity
   - Execute - Account Entity
   - Execute - AccountPerson Entity
1. Customize the request body as needed.
1. Click Send.
1. Verify the response:
   - HTTP 201 Created
   - Response contains `requestId`

### Test Callback Response

Verify Adobe accepts your callback responses.

1. After your service processes a request, select Callback - Success or Callback - Error.
1. Update `callbackToken` to match the token from your execution request.
1. Click Send.
1. Verify HTTP 200 OK.

## Advanced Features

### Dynamic Callback Tokens

Add a pre-request script to generate unique callback tokens for testing:

```javascript
// Pre-request Script
pm.environment.set("callbackToken", pm.variables.replaceIn('{{$randomUUID}}'));
console.log("Generated callbackToken:", pm.environment.get("callbackToken"));
```

### Response Validation Tests

Add test scripts to validate responses automatically:

```javascript
// Tests Tab
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response has required fields", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('status');
    pm.expect(jsonData).to.have.property('requestId');
});

pm.test("Response time is acceptable", function () {
    pm.expect(pm.response.responseTime).to.be.below(5000);
});
```

## Authentication

### API Key Authentication

Set in Headers:

```
X-API-Key: {{apiKey}}
```

### OAuth 2.0

Set in Authorization:

- Type: Bearer Token
- Token: `{{bearerToken}}`

### Basic Authentication

Set in Authorization:

- Type: Basic Auth
- Username: `{{username}}`
- Password: `{{password}}`

## Testing Scenarios

### Scenario 1: Lead Enrichment

1. Get Service Definition - Verify lead entity support.
1. Execute - Lead Entity - Send lead data.
1. Callback - Success - Return enriched data with accessor values.

### Scenario 2: Error Handling

1. Execute - Lead Entity - Send invalid data.
1. Callback - Error - Return error details.

### Scenario 3: Split Path Decisioning

1. Execute - Lead Entity - Send lead data.
1. Callback - Success - Return accessor values for journey routing.

