---
title: OpenAPI Spec Requirements
---

# Service Provider OpenAPI Spec Requirements

The first step in integrating with Adobe External Actions is to create an OpenAPI 3.0.x specification that defines your service. This specification serves as the contract between your service and Adobe, declaring your capabilities, authentication methods, and the required endpoints that enable the integration.

## Required Components

Your OpenAPI specification must include three essential components:

### OpenAPI Version

Your spec must use OpenAPI 3.0.x:

```yaml
openapi: 3.0.0
```

### Three Required Endpoints

Your service must implement these three endpoints:

| Endpoint | Method | Purpose |
| --- | --- | --- |
| `/getServiceDefinition` | GET | Returns your service capabilities and configuration |
| `/submitAsyncAction` | POST | Receives execution requests from Adobe |
| `/status` | GET | Status check endpoint for monitoring |

- `/getServiceDefinition`: Declares what entity types you support, what data you need, and what data you can return. See the [Service Definition Guide](/docs/service-definition/) for complete details.
- `/submitAsyncAction`: Receives entity data from Adobe for processing. Must return `201 Accepted` immediately and process asynchronously. See [Execution Request](/docs/execution-request/) for details.
- `/status`: Simple status check that returns `200 OK` with a status indicator. Used by Adobe for service monitoring.

### Security Scheme

Your spec must define at least one of the following allowed authentication methods:

#### Option 1: API Key Authentication

```yaml
components:
  securitySchemes:
    apiKey:
      type: apiKey
      in: header          # or 'query'
      name: X-API-Key     # customize header/query param name
      description: API Key authentication

security:
  - apiKey: []
```

- Set `in` to `header` or `query` based on your preference
- Change `name` to specify your custom header or query parameter name

#### Option 2: OAuth 2.0

```yaml
components:
  securitySchemes:
    oauth2:
      type: oauth2
      description: OAuth2 authentication
      flows:
        # Authorization Code Flow
        authorizationCode:
          authorizationUrl: https://your-service.com/oauth/authorize
          tokenUrl: https://your-service.com/oauth/token
          refreshUrl: https://your-service.com/oauth/refreshToken
          scopes: {}
          x-grantTypeName: Grant type name for access token request
          x-clientIdName: Client ID name for access token request
          x-clientSecretName: Client secret name for access token request

        # OR Client Credentials Flow
        clientCredentials:
          tokenUrl: https://your-service.com/oauth/token
          refreshUrl: https://your-service.com/oauth/refreshToken
          scopes: {}

security:
  - oauth2: []
```

- Scopes must be empty (`{}`)
- Use `x-fields` to define non-standard parameter names for token requests
- Choose either authorization code or client credentials flow based on your authentication model

#### Option 3: HTTP Basic Authentication

```yaml
components:
  securitySchemes:
    basicAuth:
      type: http
      scheme: basic
      description: HTTP Basic authentication

security:
  - basicAuth: []
```

## Important Notes

- Adobe only allows one security scheme defined in your root-level `security` array
- The security scheme must be both:
  1. Defined in `components.securitySchemes`
  1. Referenced in the root-level `security` array

## Complete Example

Here's a minimal but complete OpenAPI spec structure:

```yaml
openapi: 3.0.0
info:
  title: My External Action Service
  version: 1.0.0
  description: Lead enrichment service with scoring capabilities
  contact:
    name: Support Team
    email: support@myservice.com

servers:
  - url: https://api.myservice.com/v1
    description: Production server

security:
  - apiKey: []

paths:
  /getServiceDefinition:
    get:
      summary: Get Service Definition
      operationId: getServiceDefinition
      tags:
        - Service Configuration
      responses:
        '200':
          description: Service definition retrieved successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ServiceDefinition'
        '401':
          description: Authentication failed
        '500':
          description: Internal server error

  /submitAsyncAction:
    post:
      summary: Execute External Action
      operationId: submitAsyncAction
      tags:
        - Action Execution
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/ExecutionRequest'
      responses:
        '201':
          description: Request accepted for processing
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ExecutionResponse'
        '400':
          description: Invalid request
        '401':
          description: Authentication failed
        '500':
          description: Internal server error

  /status:
    get:
      summary: Status Check
      operationId: statusCheck
      tags:
        - Monitoring
      responses:
        '200':
          description: Service is operational
          content:
            application/json:
              schema:
                type: object
                properties:
                  status:
                    type: string
                    enum: [healthy]
                  timestamp:
                    type: string
                    format: date-time

components:
  securitySchemes:
    apiKey:
      type: apiKey
      in: header
      name: X-API-Key
      description: API Key authentication

  schemas:
    ServiceDefinition:
      type: object
      # ... your service definition schema

    ExecutionRequest:
      type: object
      # ... execution request schema

    ExecutionResponse:
      type: object
      required:
        - status
        - requestId
      properties:
        status:
          type: string
          enum: [accepted]
        requestId:
          type: string
```

## Next Steps

Once your your OpenAPI specification is ready:

1. Define your service capabilities - Implement the `/getServiceDefinition` endpoint following the [Service Definition Guide](/docs/service-definition/)
1. Understand the execution flow - Review the [Execution Request](/docs/execution-request/) structure
1. Implement the callback - Learn how to return results via [Callback Response](/docs/callback-response/)
1. Review the complete data flow - See [Data Flow](/docs/data-flow/) for the end-to-end integration flow

## Resources

- [Canonical OpenAPI Specification](/openapi/provider-canonical-openapi.yaml) - The source of truth
- [Service Definition Guide](/docs/service-definition/) - Detailed endpoint requirements
- [Examples](/docs/examples/) - Complete working examples for all entity types
- [OpenAPI 3.0 Specification](https://swagger.io/specification/) - Official OpenAPI docs
