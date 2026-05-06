---
title: Error Handling
description: Error model, response patterns, and best practices for robust External Actions integrations.
hideBreadcrumbs: true
---

# Error Handling

Proper error handling is critical for debugging, monitoring, and providing a good integration experience.
This document covers the three distinct error flows in External Actions integrations:

1. **Your service → Adobe** — synchronous HTTP errors from `/submitAsyncAction` when your service cannot accept a request
2. **Your service → Adobe** — callback payload errors sent to `callbackUrl` after asynchronous processing
3. **Adobe → Your service** — error responses from Adobe's callback endpoint when your callback POST fails

---

## 1. Errors from Your Service (`/submitAsyncAction`)

When Adobe calls your `/submitAsyncAction` endpoint and your service cannot accept the request, return a synchronous HTTP error. 



### 400 Bad Request

```json
{
  "status": "error",
  "code": "INVALID_REQUEST",
  "message": "Missing required field: email",
  "details": [
    {
      "field": "objectData[0].objectContext.leadData.email",
      "message": "Field is required"
    }
  ]
}
```

Causes:

* Malformed JSON
* Missing required fields
* Invalid data types
* Schema validation failures

### 401 Unauthorized

```json
{
  "status": "error",
  "code": "UNAUTHORIZED",
  "message": "Invalid or missing authentication credentials"
}
```

Causes:

* Missing authentication headers
* Invalid API keys
* Expired tokens



## 2. Callback Error Payloads

After accepting the execution request (HTTP 200/201), your service processes the data asynchronously and POSTs results to the `callbackUrl`. There are two ways to report errors in the callback.

### Top-Level Callback Error

Use this when your service cannot complete processing at all — for example, due to a timeout or system failure. Report the failure at the top level with an empty `objectData` array:

```json
{
  "errorCode": "TIMEOUT",
  "errorMessage": "Processing exceeded the configured timeout limit",
  "objectData": []
}
```

| Field | Type | Description |
| --- | --- | --- |
| `errorCode` | string | Stable error code identifying the failure type |
| `errorMessage` | string | Human-readable description of the failure |
| `objectData` | array | Must be present as an empty array |

### Per-Entity Callback Error

Use this when processing fails for a specific entity but the callback can still be sent. Report the failure inside `activityData` within the `objectData` entry:

```json
{
  "objectData": [
    {
      "activityData": {
        "success": false,
        "errorCode": "INVALID_DATA",
        "reason": "External API not able to process data"
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
* The entity ID (e.g. `leadData.id`) even in failure cases

### Error Code Reference

Use meaningful, consistent error codes across both top-level and per-entity errors:

| Code | Description | Use Case |
| --- | --- | --- |
| `TIMEOUT` | Processing timeout | Service took too long |
| `RATE_LIMIT_EXCEEDED` | Rate limit hit | Too many downstream API calls |
| `INVALID_DATA` | Invalid input data | Data validation error |
| `NOT_FOUND` | Entity not found | No matching record in external system |

---

## 3. Adobe Callback Endpoint Errors

When your service POSTs results to Adobe's `callbackUrl`, Adobe's endpoint may return an error. All Adobe error responses follow this format:

```json
{
  "title": "ErrMissingOauthToken",
  "status": 403,
  "error_code": 403010,
  "message": "Oauth token is missing"
}
```

| Field | Type | Description |
| --- | --- | --- |
| `title` | string | Error name/identifier |
| `status` | integer | HTTP status code |
| `error_code` | integer | Adobe-specific numeric error code |
| `message` | string | Human-readable error description |

### 400 Bad Request

Causes:

* Invalid or expired `X-Callback-Token`
* Malformed callback payload
* Schema validation failure



### 401 Unauthorized

Causes:

* Missing `Authorization` bearer token
* Invalid or expired bearer token
* Missing `X-Api-Key` header

Response:

* Refresh your OAuth Server-to-Server access token via Adobe Developer Console
* Verify all required headers are present (`Authorization`, `X-Api-Key`, `X-Request-Id`, `x-gw-ims-org-id`, `X-Callback-Token`)
* Do not retry without fixing authentication

### 403 Forbidden

Causes:

* `x-gw-ims-org-id` does not match the OAuth token's organization
* Organization not authorized for External Actions

Response:

* Verify your IMS Organization ID
* Check Developer Console permissions and OAuth credential scope

### 429 Rate Limit Exceeded

Response:

* Implement retry logic with exponential backoff


### 500 Internal Server Error

Causes:

* Adobe service temporarily unavailable
* Internal processing error

Response:

* Implement retry logic with exponential backoff
* Max 3–5 retries before alerting

---

## Best Practices

* **Always echo entity IDs** — Include required IDs (`leadData.id`, `accountData.id`, etc.) in every callback entry, even failed ones, so Adobe can correlate results
* **Use stable error codes** — Error codes should be consistent identifiers, not free-form strings; Adobe uses them for alerting and routing
* **Distinguish error levels** — Use top-level `errorCode`/`errorMessage` for system-wide failures; use per-entity `activityData` errors for partial failures
* **Log full Adobe error responses** — The `title`, `error_code`, and `message` fields from Adobe are needed for support and debugging
* **Respond quickly to Adobe** — Return HTTP 200/201 from `/submitAsyncAction` immediately; process asynchronously and callback later
