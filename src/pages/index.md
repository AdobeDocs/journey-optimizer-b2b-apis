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

Start building your External Actions API service by reading the following documentation:

See [OpenAPI Spec Requirements](openapi-spec-requirements.md) to define your OpenAPI spec.
The [Service Definition](service-definition.md) calls `GET /getServiceDefinition`. Returns capabilities, payload definitions, and split-path support.
[Execution Request](execution-request.md.md) sends `POST /submitAsyncAction` payload and responds quickly, then process asynchronously.
The [Callback Response](callback-response.md.md) receives callback at provided `callbackUrl` and returns `activityData`.
[Path Condition Accessors](path-condition-accessors.md) evaluates path conditions when split paths are enabled.
Error Lifecycle Captures request and callback failures. See [Error Handling](error-handling.md)

Download the [OpenAPI Specification](../../static/ajo-b2b-external-actions.yaml).
Download the [Postman Collection](postman.md) for API testing.
