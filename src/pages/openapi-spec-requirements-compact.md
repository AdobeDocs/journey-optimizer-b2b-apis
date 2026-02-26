---
title: OpenAPI Spec Requirements (Compact)
---

# Service Provider OpenAPI Spec Requirements (Compact)

Your OpenAPI spec is the contract Adobe uses to integrate with your service. This page lists the minimum required elements and validation checklist.

## Hard Requirements

| Requirement | Must Have |
| --- | --- |
| OpenAPI version | `openapi: 3.0.x` |
| Required paths | `GET /getServiceDefinition`, `POST /submitAsyncAction`, `GET /status` |
| Security | Exactly one root-level security scheme selected from allowed options |

## Required Endpoints

| Path | Method | Required behavior |
| --- | --- | --- |
| `/getServiceDefinition` | `GET` | Return service capabilities/configuration |
| `/submitAsyncAction` | `POST` | Accept execution requests; return `201` for async processing |
| `/status` | `GET` | Return `200` health status for monitoring |

## Allowed Security Schemes

Define one scheme in `components.securitySchemes` and reference only that scheme in root `security`.

| Scheme | OpenAPI type | Notes |
| --- | --- | --- |
| API Key | `apiKey` | `in: header` or `in: query`; custom `name` supported |
| OAuth2 | `oauth2` | Authorization code or client credentials; use empty `scopes: {}` |
| HTTP Basic | `http` + `scheme: basic` | Standard basic auth |

### Security Rule

- Adobe allows one selected scheme at root-level `security`.
- The selected scheme must exist in `components.securitySchemes`.

## Minimal Spec Skeleton

```yaml
openapi: 3.0.0
info:
  title: My External Action Service
  version: 1.0.0
servers:
  - url: https://api.myservice.com/v1
security:
  - apiKey: []
paths:
  /getServiceDefinition:
    get:
      responses:
        '200': { description: OK }
  /submitAsyncAction:
    post:
      responses:
        '201': { description: Accepted }
  /status:
    get:
      responses:
        '200': { description: Healthy }
components:
  securitySchemes:
    apiKey:
      type: apiKey
      in: header
      name: X-API-Key
```

## Recommended Response Codes

| Endpoint | Common required/expected codes |
| --- | --- |
| `/getServiceDefinition` | `200`, `401`, `500` |
| `/submitAsyncAction` | `201`, `400`, `401`, `500` |
| `/status` | `200` |

## Validation Checklist

- Spec declares OpenAPI `3.0.x`.
- All three required paths exist with correct HTTP methods.
- Root `security` selects exactly one allowed scheme.
- The selected security scheme is defined under `components.securitySchemes`.
- `/submitAsyncAction` returns `201` for accepted async requests.
- Request/response schemas align with Adobe contract.

## Next Steps

1. Define your service capabilities in [Service Definition](/docs/service-definition/).
1. Implement request handling from [Execution Request](/docs/execution-request/).
1. Implement callback payloads from [Callback Response](/docs/callback-response/).
1. Validate against [Canonical OpenAPI Specification](/openapi/provider-canonical-openapi.yaml).

## Resources

- [Canonical OpenAPI Specification](/openapi/provider-canonical-openapi.yaml)
- [OpenAPI 3.0 Specification](https://swagger.io/specification/)
