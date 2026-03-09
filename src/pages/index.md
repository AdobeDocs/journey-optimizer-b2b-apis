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

This External Actions API goes through the following flow:

1. OpenAPI contract validates your API contract and required endpoints. See [OpenAPI Spec Requirements](../openapi-spec-requirements.md)
1. Service Definition calls `GET /getServiceDefinition`. Returns capabilities, payload definitions, and split-path support. See[Service Definition Guide](../service-definition.md)
1. Execution Request sends `POST /submitAsyncAction` payload and responds quickly, then process asynchronously. See [Execution Request](../execution-request.md.md)
1. Async Processing waits for the callback while tracking action timeout. see [Execution Request](../execution-request.md.md)
1. Callback Response receives callback at provided `callbackUrl` and returns `activityData`. See [Callback Response](../callback-response.md.md)
1. Journey Routing evaluates path conditions when split paths are enabled. See [Path Condition Accessors](path-condition-accessors.md)
1. Error Lifecycle Captures request and callback failures. See [Error Handling](error-handling.md)

Download the [OpenAPI Specification](../../static/ajo-b2b-external-actions.yaml).
Download the [Postman Collection](postman.md) for API testing.
