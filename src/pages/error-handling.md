---

title: Error Handling
---

# Error Handling

Proper error handling is critical for debugging, monitoring, and providing a good integration experience. This document covers error patterns, codes, and best practices for the External Actions API.

## Error Types

### Request Errors (Execution Endpoint)

Errors that occur when Adobe sends the execution request to your service.

### Processing Errors (Internal)

Errors that occur during your service's processing logic.

### Callback Errors (Callback to Adobe)

Errors that occur when sending results back to Adobe.

## Request Error Handling

### Synchronous Response Errors

When your `/submitAsyncAction` endpoint cannot accept the request, return an HTTP error:

#### 400 Bad Request

```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Missing required field: email",
    "details": {
      "field": "objectData.objectContext.leadData.email",
      "expected": "string",
      "received": "null"
    }
  }
}
```

**Use for:**

- Malformed JSON
- Missing required fields
- Invalid data types
- Schema validation failures

#### 401 Unauthorized

```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Invalid or missing authentication credentials"
  }
}
```

**Use for:**

- Missing authentication headers
- Invalid API keys
- Expired tokens

#### 403 Forbidden

```json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "Insufficient permissions to access this resource"
  }
}
```

**Use for:**

- Valid credentials but insufficient permissions
- Service disabled for this customer

#### 429 Rate Limit Exceeded

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests",
    "retryAfter": 60
  }
}
```

**Use for:**

- Rate limiting enforcement
- Include `Retry-After` header

#### 500 Internal Server Error

```json
{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "An unexpected error occurred"
  }
}
```

**Use for:**

- Unexpected server errors
- Service unavailable
- Database errors

#### 503 Service Unavailable

```json
{
  "error": {
    "code": "SERVICE_UNAVAILABLE",
    "message": "Service is temporarily unavailable"
  }
}
```

**Use for:**

- Planned maintenance
- Temporary outages
- Circuit breaker triggered

## Error Handling

### Callback with Activity Error

When processing fails but you can still send a callback, use `activityData` to indicate the error:

```json
{
  "objectData": [
    {
      "activityData": {
        "success": false,
        "errorCode": "ENRICHMENT_FAILED",
        "reason": "External API returned 503 Service Unavailable"
      },
      "leadData": {
        "id": 12345
      }
    }
  ]
}
```

**Always include:**

- `activityData.success: false`
- A stable `errorCode`
- A clear `reason`

Use meaningful, consistent error codes. Examples:

| Code | Description | Use Case |
| --- | --- | --- |
| `RATE_LIMIT_EXCEEDED` | Rate limit hit | Too many API calls |
| `INVALID_DATA` | Invalid input data | Data validation error |
| `TIMEOUT` | Processing timeout | Took too long |
| `NOT_FOUND` | Entity not found | No matching record |
| `EXTERNAL_API_ERROR` | Third-party API error | Downstream failure |
| `AUTHENTICATION_FAILED` | Auth to external system failed | Credentials invalid |
| `INSUFFICIENT_DATA` | Not enough data to process | Missing required fields |
| `DUPLICATE_REQUEST` | Request already processed | Idempotency check failed |

## Callback Error Handling

### Adobe's Callback Endpoint Errors

When Adobe's callback endpoint returns an error:

#### 400 Bad Request

```json
{
  "error": {
    "code": "INVALID_CALLBACK",
    "message": "Invalid token"
  }
}
```

**Possible causes:**

- Invalid or expired correlation ID
- Malformed callback payload
- Schema validation failure

**Your action:**

- Log the error
- Do not retry (client error)
- Alert for manual intervention

#### 401 Unauthorized

```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Invalid or missing authentication headers"
  }
}
```

**Possible causes:**

- Missing required headers
- Invalid API key
- Expired bearer token

**Your action:**

- Check authentication configuration
- Refresh tokens if expired
- Do not retry without fixing auth

#### 500 Internal Server Error

```json
{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "An unexpected error occurred processing the callback"
  }
}
```

**Possible causes:**

- Adobe service temporarily unavailable
- Unexpected data format
- Internal processing error

**Your action:**

- Implement retry logic with exponential backoff
- Max 3-5 retries
- Alert if all retries fail


## Best Practices

### Use Specific Error Codes

```json
// Good: Specific, actionable error code
{
  "errorCode": "RATE_LIMIT_EXCEEDED",
  "reason": "API rate limit of 100 requests/minute exceeded"
}

// Bad: Generic error code
{
  "errorCode": "ERROR",
  "reason": "Something went wrong"
}
```

### Provide Actionable Error Messages

```json
// Good: Clear, actionable message
{
  "errorCode": "INVALID_EMAIL",
  "reason": "Email format is invalid: 'not-an-email'"
}

// Bad: Vague message
{
  "errorCode": "BAD_DATA",
  "reason": "Data is bad"
}
```

### Always Echo Required IDs

Even in error cases, always include required entity IDs:

```json
{
  "activityData": {
    "success": false,
    "errorCode": "ENRICHMENT_FAILED"
  },
  "leadData": {
    "id": 12345  // Always include
  }
}
```

Maintain a comprehensive error code reference:

```yaml
errorCodes:
  ENRICHMENT_FAILED:
    description: Data enrichment operation failed
    httpStatus: 200  # Callback succeeds, processing failed
    retryable: true
  INVALID_DATA:
    description: Input data validation failed
    httpStatus: 400
    retryable: false
  RATE_LIMIT_EXCEEDED:
    description: API rate limit exceeded
    httpStatus: 429
    retryable: true
```

## Troubleshooting Guide

| Symptom | Possible Cause | Solution |
| --- | --- | --- |
| All requests failing | Service down / configuration error | Check service health, validate config |
| Callbacks not received | Wrong callback URL / network issue | Verify callback URL, check network |
| Intermittent failures | Rate limiting / timeouts | Implement backoff, increase timeout |
| Invalid correlation ID | Callback delayed / ID not stored | Store IDs, check timing |
| Field not updating | Field not in `callbackPayloadDef` | Add field to service definition |
| Token values ignored | `enableSplitPaths` not set | Enable split paths in service definition |

## Next Steps

- Review [Execution Request](/docs/execution-request/) for request validation
- Review [Callback Response](/docs/callback-response/) for callback structure
- See [Examples](/docs/examples/) for error handling examples
