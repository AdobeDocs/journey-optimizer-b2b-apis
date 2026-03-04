---
title: External Actions Data Flow
description: End-to-end integration flow for service definition, execution requests, processing, and callbacks.
---

# External Actions Data Flow

The Adobe External Actions API enables external services to integrate with Adobe Journey Optimizer B2B Edition through custom journey actions.

## End-to-End Sequence

| Step | Adobe Action | Service Provider Action | Detailed Reference |
| --- | --- | --- | --- |
| 1. OpenAPI Contract | Validates your API contract and required endpoints | Publish OpenAPI 3.0.x spec with required endpoints and security | [OpenAPI Spec Requirements](../openapi-spec-requirements.md) |
| 2. Service Definition | Calls `GET /getServiceDefinition` | Return capabilities, payload definitions, and split-path support | [Service Definition Guide](../service-definition.md) |
| 3. Execution Request | Sends `POST /submitAsyncAction` payload | Accept request and respond quickly, then process asynchronously | [Execution Request](execution-request.md) |
| 4. Async Processing | Waits for callback while tracking action timeout | Execute enrichment/scoring/decision logic | [Execution Request](execution-request.md) |
| 5. Callback Response | Receives callback at provided `callbackUrl` | Return `activityData` and optional entity updates | [Callback Response](callback-response.md) |
| 6. Journey Routing | Evaluates path conditions when split paths are enabled | Provide `accessorValues` for routing decisions | [Path Condition Accessors](path-condition-accessors.md) |
| 7. Error Lifecycle | Captures request and callback failures | Return clear errors and retry only when appropriate | [Error Handling](error-handling.md) |

## Integration Checklist

| Area | Requirement |
| --- | --- |
| Contract | OpenAPI 3.0.x with `/getServiceDefinition`, `/submitAsyncAction`, `/status` |
| Security | At least one of `apiKey`, `oauth2`, or `basicAuth` |
| Entity model | Support exactly one entity type: `lead`, `account`, or `accountPerson` |
| Async behavior | Accept execution request immediately and process out-of-band |
| Callback | Send required IDs and `activityData` in callback payload |
| Split paths | If enabled, provide accessor values matching declared metadata |

## Data Handoff Points

| Handoff | Payload Owner | Description |
| --- | --- | --- |
| Service definition fields | Service provider | Declares what Adobe can send and what your callback can update |
| Execution payload | Adobe | Sends mapped entity data and runtime context |
| Callback payload | Service provider | Returns processing outcome and optional field updates |

## Field Mapping Lifecycle

1. Service provider declares callback-capable fields in service definition.
1. Admin maps those fields in Adobe configuration.
1. Adobe sends mapped values during execution.
1. Service returns only fields it wants Adobe to update.

## Next References

- [Service Definition Guide](../service-definition.md)
- [Execution Request](execution-request.md)
- [Callback Response](callback-response.md)
- [Path Condition Accessors](path-condition-accessors.md)
- [Error Handling](error-handling.md)
