---
title: Adobe Journey Optimizer B2B External Actions API
description: Adobe Journey Optimizer B2B External Actions API Specification
---

# Adobe Journey Optimizer B2B External Actions API

> ⚠️ **Beta Release**: This specification is a beta release. Endpoints, schemas, and behavior may change without notice.

The External Actions API allows you to connect your service provider endpoints to Adobe Journey Optimizer, enabling you to execute custom actions as part of customer journeys.

- Asynchronous Processing - Adobe sends requests, you process them asynchronously
- Flexible Entity Types - Support for leads, accounts, and more
- Rich Data Mapping - Map Adobe data to your service fields
- Error Handling - Robust error reporting mechanisms
- Security - Multiple authentication options (API Key, Basic, OAuth2)

## API Reference

- [Download OpenAPI Specification](../../static/ajo-b2b-external-actions.yaml) - Complete API specification in OpenAPI 3.0 format

## Quick Links

- [OpenAPI Spec Requirements](openapi-spec-requirements.md)
- [Data Flow](data-flow.md)
- [Service Definition](service-definition.md)
- [Execution Request](execution-request.md)
- [Callback Response](callback-response.md)
- [Error Handling](error-handling.md)
- [Path & Condition Accessors](path-condition-accessors.md)
- [Examples](examples.md)
- [Postman Collection](postman.md)

> ℹ️ **Note**: Support for person audiences will arrive in a follow-up release with Person Journeys.
