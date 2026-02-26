---

title: Postman Collection
---

# Postman Collection

The Postman collection provides a complete set of pre-configured requests for testing all API endpoints, including service discovery, execution requests, and callback responses.

## Download Collection

📦 **[Download Postman Collection](https://github.com/AdobeDocs/ajo-external-actions-api-spec/blob/main/postman/External-Actions-API.postman_collection.json)**

## Collection Contents

The collection includes requests for:

- Service Definition - Get service capabilities and configuration
- Execution Requests - Test entity submissions (lead, account, `accountPerson`)
- Callback Responses - Test callback submissions to Adobe
- Health Check - Verify service availability

## Quick Setup

### Import Collection

1. Open Postman Desktop or Web.
1. Click Import.
1. Select `External-Actions-API.postman_collection.json`.
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

### Test Your First Request

1. Select Get Service Definition request.
1. Ensure your environment is selected.
1. Click Send.
1. Verify the response matches your service definition.

## Environment Setup

### Create Environment File

You can create a Postman environment JSON file:

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

### Multiple Environments

Create separate environments for different stages:

- Development - Local or dev server
- Staging - Staging environment
- Production - Production environment (use with caution!)

## Testing Workflows

### Test Service Definition

**Purpose:** Verify your service declares capabilities correctly.

1. Select Get Service Definition
1. Click Send
1. Verify response contains:
   - `supportedEntityType`
   - `invocationPayloadDef`
   - `callbackPayloadDef`
   - `enableSplitPaths` (if applicable)

### Test Execution Request

**Purpose:** Verify your service accepts and processes execution requests.

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

**Purpose:** Verify Adobe accepts your callback responses.

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

### Chain Requests

Use Collection Runner or Newman to chain requests:

1. Get Service Definition.
1. Submit Execution Request (save correlation ID).
1. Wait for processing.
1. Send Callback Response (use saved correlation ID).

### Mock Server Setup

Create a Postman Mock Server to test callbacks:

1. Right-click collection → Mock Collection.
1. Create mock server.
1. Copy the mock URL.
1. Use as your `callbackUrl` in execution requests.
1. View incoming callbacks in Postman.

## Authentication

The collection supports multiple authentication methods:

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

## Command Line Testing (Newman)

Run the collection from the command line using Newman:

### Install Newman

```bash
npm install -g newman
```

### Run Collection

```bash
# Run with environment file
newman run External-Actions-API.postman_collection.json \
  -e dev-environment.json

# Run with specific folder
newman run External-Actions-API.postman_collection.json \
  --folder "Execution Requests"

# Generate HTML report
newman run External-Actions-API.postman_collection.json \
  -e dev-environment.json \
  --reporters cli,html \
  --reporter-html-export report.html
```

### CI/CD Integration

Add the following to your CI/CD pipeline:

```yaml
# GitHub Actions example
- name: Run API Tests
  run: |
    npm install -g newman
    newman run postman/External-Actions-API.postman_collection.json \
      -e postman/dev-environment.json \
      --reporters cli,json \
      --reporter-json-export test-results.json
```

## Troubleshooting

### Common Issues

| Issue | Solution |
| --- | --- |
| **401 Unauthorized** | Check API key in environment variables |
| **400 Bad Request** | Validate request body against OpenAPI spec |
| **Connection Error** | Verify service URL is accessible |
| **Timeout** | Increase timeout in Postman settings (File → Settings → General) |
| **CORS Error** | Disable Postman interceptor or use desktop app |

### Debugging Tips

1. Enable Console - View → Show Postman Console
1. Check Variables - Hover over `{{variable}}` to see resolved values
1. Validate JSON - Use online JSON validators for request bodies
1. Check Headers - Ensure all required headers are present
1. Review Logs - Check your service logs for detailed errors

## Best Practices

### Organization

- Use Folders - Group related requests
- Name Clearly - Use descriptive request names
- Document - Add descriptions to requests
- Version Control - Store collection in Git

### Variables

- Environment Variables - Use for URLs, API keys
- Collection Variables - Use for shared test data
- Global Variables - Avoid for security reasons

### Security

- Never commit secrets - Use `.gitignore` for environment files
- Use environment variables - Don't hardcode API keys
- Separate environments - Different credentials per stage
- Rotate credentials - Regularly update API keys

## Related Resources

- [API Reference](/api) - Interactive OpenAPI documentation
- [Examples](/docs/examples/) - Complete request/response examples
- [Service Definition](/docs/service-definition/) - Service configuration guide
- [Execution Request](/docs/execution-request/) - Request structure details
- [Callback Response](/docs/callback-response/) - Response format guide

## Support

For issues or questions:

- Review the [OpenAPI specification](https://github.com/AdobeDocs/ajo-external-actions-api-spec/blob/main/static/openapi/provider-canonical-openapi.yaml)
- Check [GitHub Issues](https://github.com/AdobeDocs/ajo-external-actions-api-spec/issues)
- Contact Adobe Support via [Experience League](https://experienceleague.adobe.com/)
