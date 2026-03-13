---
title: Journey Optimizer B2B External Actions API
description: The Journey Optimizer B2B External Actions API allows you to integrate your custom service with Adobe directly.
---

<Superhero slots="heading, text" />

# Journey Optimizer B2B External Actions API

The Journey Optimizer B2B External Actions API enables external services to integrate with Journey Optimizer B2B Edition through custom journey actions.

**Beta Release**: This specification is a beta release.
Endpoints, schemas, and behavior may change without notice.

The External Actions API enables:

- Asynchronous Processing - Adobe sends requests, you process them asynchronously
- Flexible Entity Types - Support for leads, accounts, and more
- Rich Data Mapping - Map Adobe data to your service fields
- Error Handling - Robust error reporting mechanisms
- Security - Multiple authentication options (API Key, Basic, OAuth2)

Start building your External Actions API service with the following documentation:

- Begin with [OpenAPI Spec Requirements](openapi-spec-requirements.md) to define your OpenAPI spec.
- The [Service Definition](service-definition.md) calls `GET /getServiceDefinition`. Returns capabilities, payload definitions, and split-path support.
- [Execution Request](execution-request.md.md) sends `POST /submitAsyncAction` payload and responds quickly, then process asynchronously.
- The [Callback Response](callback-response.md.md) receives callback at provided `callbackUrl` and returns `activityData`.
- [Path Condition Accessors](path-condition-accessors.md) evaluates path conditions when split paths are enabled.
- Error Lifecycle Captures request and callback failures. See [Error Handling](error-handling.md)

Download the [OpenAPI Specification](../../static/ajo-b2b-external-actions.yaml) and the [Postman Collection](postman.md) for API testing.

