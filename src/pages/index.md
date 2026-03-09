---
title: Adobe Journey Optimizer B2B External Actions API
description: Adobe Journey Optimizer B2B External Actions API Specification
---

# Adobe Journey Optimizer B2B External Actions API

> ⚠️ **Beta Release**: This specification is a beta release. Endpoints, schemas, and behavior may change without notice.

The Adobe External Actions API enables external services to integrate with Adobe Journey Optimizer B2B Edition through custom journey actions. This guide explains how to write and configure your service to receive data from Adobe, and return the processed data back to your journey.

- Asynchronous Processing - Adobe sends requests, you process them asynchronously
- Flexible Entity Types - Support for leads, accounts, and more
- Rich Data Mapping - Map Adobe data to your service fields
- Error Handling - Robust error reporting mechanisms
- Security - Multiple authentication options (API Key, Basic, OAuth2)


To get started with the External Actions API, read through this guide completely.

| Step | Adobe Action | Service Provider Action | Reference |
| --- | --- | --- | --- |
| 1. OpenAPI Contract | Validates your API contract and required endpoints | Publish OpenAPI 3.0.x spec with required endpoints and security | [OpenAPI Spec Requirements](../openapi-spec-requirements.md) |
| 2. Service Definition | Calls `GET /getServiceDefinition` | Return capabilities, payload definitions, and split-path support | [Service Definition Guide](../service-definition.md) |
| 3. Execution Request | Sends `POST /submitAsyncAction` payload | Accept request and respond quickly, then process asynchronously | [Execution Request](../execution-request.md.md) |
| 4. Async Processing | Waits for callback while tracking action timeout | Execute enrichment/scoring/decision logic | [Execution Request](../execution-request.md.md) |
| 5. Callback Response | Receives callback at provided `callbackUrl` | Return `activityData` and optional entity updates | [Callback Response](../callback-response.md.md) |
| 6. Journey Routing | Evaluates path conditions when split paths are enabled | Provide `accessorValues` for routing decisions | [Path Condition Accessors](path-condition-accessors.md) |
| 7. Error Lifecycle | Captures request and callback failures | Return clear errors and retry only when appropriate | [Error Handling](error-handling.md) |


Download the [OpenAPI Specification](../../static/ajo-b2b-external-actions.yaml).
