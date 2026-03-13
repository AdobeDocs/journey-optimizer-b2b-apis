---
title: Error Handling
description: Error model, response patterns, and best practices for robust External Actions integrations.
---

# Error Handling

Proper error handling is critical for debugging, monitoring, and providing a good integration experience. This document covers error patterns, codes, and best practices for the External Actions API.

## Error Types

* Request errors occur when Adobe sends the execution request to your service.
* Processing errors occur during your service's processing logic.
* Callback errors that occur when sending results back to Adobe.
* Synchronous response errors happen when your `/submitAsyncAction` endpoint cannot accept the request and returns an HTTP error.

### 400 Bad Request

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

Causes may be due to:

* Malformed JSON
* Missing required fields
* Invalid data types
* Schema validation failures

### 401 Unauthorized

```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Invalid or missing authentication credentials"
  }
}
```

Causes may be due to:

* Missing authentication headers
* Invalid API keys
* Expired tokens

### 403 Forbidden

```json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "Insufficient permissions to access this resource"
  }
}
```

Causes may be due to:

* Valid credentials but insufficient permissions
* Service disabled for this customer

### 429 Rate Limit Exceeded

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests",
    "retryAfter": 60
  }
}
```

Causes may be due to:

* Rate limiting enforcement
* Include `Retry-After` header

### 500 Internal Server Error

```json
{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "An unexpected error occurred"
  }
}
```

Causes may be due to:

* Unexpected server errors
* Service unavailable
* Database errors

### 503 Service Unavailable

```json
{
  "error": {
    "code": "SERVICE_UNAVAILABLE",
    "message": "Service is temporarily unavailable"
  }
}
```

Causes may be due to:

* Planned maintenance
* Temporary outages
* Circuit breaker triggered

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

Always include:

* `activityData.success: false`
* A stable `errorCode`
* A clear `reason`

Use meaningful, consistent error codes. For example:

| Code | Description | Use Case |
| -- | -- | -- |
| `RATE_LIMIT_EXCEEDED` | Rate limit hit | Too many API calls |
| `INVALID_DATA` | Invalid input data | Data validation error |
| `TIMEOUT` | Processing timeout | Took too long |
| `NOT_FOUND` | Entity not found | No matching record |
| `EXTERNAL_API_ERROR` | Third-party API error | Downstream failure |
| `AUTHENTICATION_FAILED` | Auth to external system failed | Credentials invalid |
| `INSUFFICIENT_DATA` | Not enough data to process | Missing required fields |
| `DUPLICATE_REQUEST` | Request already processed | Idempotency check failed |

## Callback Error Handling

### Adobe Callback Endpoint Errors

When Adobe's callback endpoint returns an error, you will see:

#### 400 Bad Request

```json
{
  "error": {
    "code": "INVALID_CALLBACK",
    "message": "Invalid token"
  }
}
```

Possible causes are:

* Invalid or expired correlation ID
* Malformed callback payload
* Schema validation failure

If you see this error:

* Log the error
* Do not retry (client error)
* Alert for manual intervention

#### 401 Unauthorized

```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Invalid or missing authentication headers"
  }
}
```

Possible causes are:

* Missing required headers
* Invalid API key
* Expired bearer token

Next steps are:

* Check authentication configuration
* Refresh tokens if expired
* Do not retry without fixing auth

#### 500 Internal Server Error

```json
{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "An unexpected error occurred processing the callback"
  }
}
```

Possible causes are:

* Adobe service temporarily unavailable
* Unexpected data format
* Internal processing error

To resolve, try:

* Implement retry logic with exponential backoff
* Max 3-5 retries
* Alert if all retries fail


## Best Practices

### Always Echo Required IDs

Even in error cases, always include required entity IDs:

```json
{
  "activityData": {
    "success": false,
    "errorCode": "ENRICHMENT_FAILED"
  },
  "leadData": {
    "id": 12345  // Always include id
  }
}
```

### Maintain a comprehensive error code reference:

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

