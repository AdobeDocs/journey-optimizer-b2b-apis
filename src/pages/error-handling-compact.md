---
title: Error Handling
---

# Error Handling

Use this page as a quick reference for handling errors in the External Actions flow.

## Error Surfaces

| Surface | Where it happens | What to return |
| --- | --- | --- |
| Request acceptance | Adobe calls your `POST /submitAsyncAction` endpoint | HTTP error status + compact error payload |
| Processing result | Your service processed the request but failed business logic | Callback with `activityData.success: false` |
| Callback delivery | Your callback call to Adobe fails | Retry or stop based on Adobe's HTTP status |

##  Request Acceptance Errors (`/submitAsyncAction`)

Return an HTTP error when the request cannot be accepted.

| HTTP Status | `error.code` | Used when | Retry by Adobe |
| --- | --- | --- | --- |
| `400` | `INVALID_REQUEST` | Malformed JSON, missing required fields, schema/type validation failure | No |
| `401` | `UNAUTHORIZED` | Missing/invalid auth credentials | No |
| `403` | `FORBIDDEN` | Credentials valid but not permitted for this resource/customer | No |
| `429` | `RATE_LIMIT_EXCEEDED` | Rate limit exceeded | Yes, after `Retry-After` |
| `500` | `INTERNAL_ERROR` | Unexpected server failure | Yes |
| `503` | `SERVICE_UNAVAILABLE` | Temporary outage/maintenance/circuit breaker | Yes |

Minimal error body format:

```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Missing required field: email"
  }
}
```

## Processing Errors in Callback (`activityData`)

If processing fails but callback delivery is possible, return success at the HTTP transport level and set:

- `activityData.success: false`
- stable `activityData.errorCode`
- clear `activityData.reason`

| `activityData.errorCode` | Recommended `reason` style | Typical case | Retryable |
| --- | --- | --- | --- |
| `RATE_LIMIT_EXCEEDED` | Include limit window and retry hint | Downstream API throttled | Yes |
| `INVALID_DATA` | Name invalid field/value | Input validation failed | No |
| `TIMEOUT` | Include timeout threshold reached | Processing exceeded SLA | Yes |
| `NOT_FOUND` | Identify missing entity type/key | Referenced record does not exist | No |
| `EXTERNAL_API_ERROR` | Include downstream status/system | Third-party dependency failure | Yes |
| `AUTHENTICATION_FAILED` | Identify auth mechanism that failed | Credential/token failure to external system | Usually No |
| `INSUFFICIENT_DATA` | List required missing fields | Not enough data to process | No |
| `DUPLICATE_REQUEST` | Reference idempotency key/request id | Request already processed | No |

Compact callback failure example:

```json
{
  "objectData": [
    {
      "activityData": {
        "success": false,
        "errorCode": "EXTERNAL_API_ERROR",
        "reason": "CRM API returned 503; retry after 60s"
      },
      "leadData": {
        "id": 12345
      }
    }
  ]
}
```

## 3) Callback Delivery Errors (Calling Adobe Callback URL)

When Adobe responds with an error to your callback request:

| Adobe HTTP Status | Typical `error.code` | Common reason | Your action | Retry |
| --- | --- | --- | --- | --- |
| `400` | `INVALID_CALLBACK` | Invalid token/correlation, malformed payload, schema mismatch | Log and fix payload/token handling | No |
| `401` | `UNAUTHORIZED` | Missing/invalid callback auth headers/tokens | Refresh/fix auth config, then retry once fixed | No (until fixed) |
| `500` | `INTERNAL_ERROR` | Adobe temporary internal issue | Exponential backoff, max 3-5 attempts | Yes |

## Compact Best Practices

| Practice | Why it matters |
| --- | --- |
| Use specific codes (not `ERROR`) | Enables routing, analytics, and deterministic handling |
| Keep reason actionable | Speeds triage and reduces support time |
| Always echo required IDs (`leadData.id`, `accountData.id`, etc.) | Prevents entity correlation issues in failure paths |
| Separate retryable vs non-retryable failures | Avoids noisy retries and duplicate processing |

## Troubleshooting Quick Map

| Symptom | Likely cause | First check |
| --- | --- | --- |
| All requests failing | Service/config outage | Service health + auth config |
| Callback rejected | Token/header/payload issue | Required headers and schema |
| Intermittent failures | Throttling/timeouts | Retry policy + downstream limits |
| Field updates missing | Field not in callback payload definition | `callbackPayloadDef` mapping |

## Next Steps

- Review [Execution Request](/docs/execution-request/) for request validation
- Review [Callback Response](/docs/callback-response/) for callback shape and required headers
- See [Examples](/docs/examples/) for end-to-end payloads
