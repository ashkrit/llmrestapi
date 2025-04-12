# REST-based Model Context Protocol API Specification

## Overview

This specification defines a REST-based implementation of the Model Context Protocol for exchanging context data, tools, prompts, and other resources between AI systems. The API follows REST principles including resource-oriented design, standard HTTP methods, hypermedia controls, and caching mechanisms.

## Base URL

All API endpoints are relative to the base URL:

```
https://api.example.com/mcp/v1
```

## Authentication

The API supports the following authentication methods:

- OAuth 2.0 Bearer Tokens
- API Keys (via `X-API-Key` header)
- JWT tokens

Authentication details should be provided in the HTTP Authorization header:

```
Authorization: Bearer {token}
```

## Common Response Headers

All responses include:

- `Content-Type`: Specifies the format of the response
- `ETag`: Entity tag for cache validation
- `X-Request-ID`: Unique identifier for the request (for tracing)
- `Cache-Control`: Caching directives

## Resources

### Server Capabilities

#### Get Server Capabilities

```
GET /capabilities
```

**Response**

```json
{
  "serverInfo": {
    "name": "Example MCP Server",
    "version": "1.0.0",
    "protocolVersion": "2024-11-05"
  },
  "capabilities": {
    "resources": {
      "subscribe": true,
      "listChanged": true
    },
    "tools": {
      "listChanged": true
    },
    "prompts": {
      "listChanged": true
    },
    "logging": {}
  },
  "instructions": "This server provides access to code repository context and tool execution.",
  "_links": {
    "self": {"href": "/capabilities"},
    "resources": {"href": "/resources"},
    "tools": {"href": "/tools"},
    "prompts": {"href": "/prompts"}
  }
}
```

### Resources

#### List Resources

```
GET /resources
```

**Query Parameters**

- `limit` (integer): Maximum number of resources to return
- `cursor` (string): Pagination cursor for fetching next page

**Response**

```json
{
  "resources": [
    {
      "id": "repo1",
      "uri": "file:///path/to/repo",
      "name": "Main Repository",
      "description": "Primary code repository",
      "mimeType": "application/x-directory",
      "size": 1048576,
      "_links": {
        "self": {"href": "/resources/repo1"},
        "content": {"href": "/resources/repo1/content"}
      }
    }
  ],
  "nextCursor": "eyJsYXN0SWQiOiJyZXBvMSJ9",
  "_links": {
    "self": {"href": "/resources?limit=10"},
    "next": {"href": "/resources?limit=10&cursor=eyJsYXN0SWQiOiJyZXBvMSJ9"}
  }
}
```

#### Get Resource

```
GET /resources/{resourceId}
```

**Response**

```json
{
  "id": "repo1",
  "uri": "file:///path/to/repo",
  "name": "Main Repository",
  "description": "Primary code repository",
  "mimeType": "application/x-directory",
  "size": 1048576,
  "annotations": {
    "audience": ["user", "assistant"],
    "priority": 0.8
  },
  "_links": {
    "self": {"href": "/resources/repo1"},
    "content": {"href": "/resources/repo1/content"}
  }
}
```

#### Get Resource Content

```
GET /resources/{resourceId}/content
```

**Headers**

- `Accept`: Specifies the preferred format for the response
- `Range`: For requesting partial content (large resources)

**Response Headers**

- `Content-Type`: Format of the returned content
- `Content-Length`: Size of the response in bytes
- `Accept-Ranges`: Indicates support for partial content requests

**Response**

For text resources:
```
Status: 200 OK
Content-Type: text/plain

This is the content of the resource.
```

For binary resources:
```
Status: 200 OK
Content-Type: application/octet-stream

[Binary data]
```

#### Create Resource

```
POST /resources
```

**Request Body**

```json
{
  "name": "User Config",
  "description": "User configuration file",
  "content": "theme=dark\nlanguage=en"
}
```

**Response**

```json
{
  "id": "config1",
  "uri": "file:///path/to/config",
  "name": "User Config",
  "description": "User configuration file",
  "mimeType": "text/plain",
  "size": 128,
  "_links": {
    "self": {"href": "/resources/config1"},
    "content": {"href": "/resources/config1/content"}
  }
}
```

#### Update Resource

```
PUT /resources/{resourceId}
```

**Request Body**

```json
{
  "name": "Updated Config",
  "description": "Updated user configuration file",
  "content": "theme=light\nlanguage=fr"
}
```

**Response**

```json
{
  "id": "config1",
  "uri": "file:///path/to/config",
  "name": "Updated Config",
  "description": "Updated user configuration file",
  "mimeType": "text/plain",
  "size": 130,
  "_links": {
    "self": {"href": "/resources/config1"},
    "content": {"href": "/resources/config1/content"}
  }
}
```

#### Subscribe to Resource Updates

```
POST /subscriptions
```

**Request Body**

```json
{
  "resourceId": "repo1",
  "events": ["updated", "deleted"],
  "callbackUrl": "https://client.example.com/webhook/resource-updates",
  "expiresIn": 3600
}
```

**Response**

```json
{
  "id": "sub123",
  "resourceId": "repo1",
  "events": ["updated", "deleted"],
  "expiresAt": "2024-05-12T15:00:00Z",
  "_links": {
    "self": {"href": "/subscriptions/sub123"},
    "resource": {"href": "/resources/repo1"},
    "events": {"href": "/subscriptions/sub123/events"}
  }
}
```

#### Stream Resource Updates

```
GET /subscriptions/{subscriptionId}/events
```

**Headers**

- `Accept: text/event-stream` (for Server-Sent Events)

**Response (Server-Sent Events)**

```
event: resource.updated
data: {"resourceId": "repo1", "timestamp": "2024-05-12T13:45:00Z"}

event: resource.deleted
data: {"resourceId": "oldfile", "timestamp": "2024-05-12T13:50:00Z"}
```

### Tools

#### List Tools

```
GET /tools
```

**Query Parameters**

- `limit` (integer): Maximum number of tools to return
- `cursor` (string): Pagination cursor

**Response**

```json
{
  "tools": [
    {
      "id": "code-search",
      "name": "Code Search",
      "description": "Search for code patterns across the repository",
      "inputSchema": {
        "type": "object",
        "properties": {
          "query": {"type": "string", "description": "Search query"},
          "filePattern": {"type": "string", "description": "File pattern to search"}
        },
        "required": ["query"]
      },
      "_links": {
        "self": {"href": "/tools/code-search"},
        "execute": {"href": "/tools/code-search/execute", "method": "POST"}
      }
    }
  ],
  "nextCursor": "eyJsYXN0SWQiOiJjb2RlLXNlYXJjaCJ9",
  "_links": {
    "self": {"href": "/tools?limit=10"},
    "next": {"href": "/tools?limit=10&cursor=eyJsYXN0SWQiOiJjb2RlLXNlYXJjaCJ9"}
  }
}
```

#### Get Tool Details

```
GET /tools/{toolId}
```

**Response**

```json
{
  "id": "code-search",
  "name": "Code Search",
  "description": "Search for code patterns across the repository",
  "inputSchema": {
    "type": "object",
    "properties": {
      "query": {"type": "string", "description": "Search query"},
      "filePattern": {"type": "string", "description": "File pattern to search"}
    },
    "required": ["query"]
  },
  "_links": {
    "self": {"href": "/tools/code-search"},
    "execute": {"href": "/tools/code-search/execute", "method": "POST"},
    "schema": {"href": "/tools/code-search/schema"}
  }
}
```

#### Execute Tool

```
POST /tools/{toolId}/execute
```

**Request Body**

```json
{
  "arguments": {
    "query": "function calculateTax",
    "filePattern": "*.js"
  }
}
```

**Response**

```json
{
  "id": "exec-567",
  "status": "success",
  "content": [
    {
      "type": "text",
      "text": "Found 3 matches in 2 files"
    },
    {
      "type": "resource",
      "resource": {
        "uri": "file:///path/to/repo/tax.js",
        "mimeType": "application/javascript",
        "text": "function calculateTax(amount, rate) {\n  return amount * rate;\n}"
      }
    }
  ],
  "_links": {
    "self": {"href": "/tools/code-search/executions/exec-567"}
  }
}
```

#### Get Tool Schema

```
GET /tools/{toolId}/schema
```

**Response**

```json
{
  "type": "object",
  "properties": {
    "query": {"type": "string", "description": "Search query"},
    "filePattern": {"type": "string", "description": "File pattern to search"}
  },
  "required": ["query"]
}
```

### Prompts

#### List Prompts

```
GET /prompts
```

**Query Parameters**

- `limit` (integer): Maximum number of prompts to return
- `cursor` (string): Pagination cursor

**Response**

```json
{
  "prompts": [
    {
      "id": "code-explanation",
      "name": "Code Explanation",
      "description": "Template for explaining code snippets",
      "arguments": [
        {
          "name": "language",
          "description": "Programming language",
          "required": true
        },
        {
          "name": "code",
          "description": "Code snippet to explain",
          "required": true
        }
      ],
      "_links": {
        "self": {"href": "/prompts/code-explanation"},
        "render": {"href": "/prompts/code-explanation/render", "method": "POST"}
      }
    }
  ],
  "nextCursor": "eyJsYXN0SWQiOiJjb2RlLWV4cGxhbmF0aW9uIn0=",
  "_links": {
    "self": {"href": "/prompts?limit=10"},
    "next": {"href": "/prompts?limit=10&cursor=eyJsYXN0SWQiOiJjb2RlLWV4cGxhbmF0aW9uIn0="}
  }
}
```

#### Get Prompt

```
GET /prompts/{promptId}
```

**Response**

```json
{
  "id": "code-explanation",
  "name": "Code Explanation",
  "description": "Template for explaining code snippets",
  "arguments": [
    {
      "name": "language",
      "description": "Programming language",
      "required": true
    },
    {
      "name": "code",
      "description": "Code snippet to explain",
      "required": true
    }
  ],
  "_links": {
    "self": {"href": "/prompts/code-explanation"},
    "render": {"href": "/prompts/code-explanation/render", "method": "POST"}
  }
}
```

#### Render Prompt

```
POST /prompts/{promptId}/render
```

**Request Body**

```json
{
  "arguments": {
    "language": "javascript",
    "code": "function add(a, b) { return a + b; }"
  }
}
```

**Response**

```json
{
  "description": "JavaScript function explanation",
  "messages": [
    {
      "role": "user",
      "content": {
        "type": "text",
        "text": "Explain this JavaScript code: function add(a, b) { return a + b; }"
      }
    }
  ],
  "_links": {
    "self": {"href": "/prompts/code-explanation/render?language=javascript"}
  }
}
```

### Sampling

#### Create Message

```
POST /sampling/messages
```

**Request Body**

```json
{
  "messages": [
    {
      "role": "user",
      "content": {
        "type": "text",
        "text": "What is the big O notation for quicksort?"
      }
    }
  ],
  "modelPreferences": {
    "intelligencePriority": 0.8,
    "speedPriority": 0.4
  },
  "systemPrompt": "You are a helpful coding assistant.",
  "includeContext": "thisServer",
  "temperature": 0.7,
  "maxTokens": 1000
}
```

**Response**

```json
{
  "role": "assistant",
  "content": {
    "type": "text",
    "text": "The average time complexity of quicksort is O(n log n)..."
  },
  "model": "example-model-2024",
  "stopReason": "endTurn",
  "_links": {
    "self": {"href": "/sampling/messages/msg123"}
  }
}
```

### Logging

#### Set Logging Level

```
POST /logging/level
```

**Request Body**

```json
{
  "level": "info"
}
```

**Response**

```json
{
  "level": "info",
  "message": "Logging level set to info",
  "_links": {
    "self": {"href": "/logging/level"}
  }
}
```

#### Stream Log Messages

```
GET /logging/stream
```

**Headers**

- `Accept: text/event-stream` (for Server-Sent Events)

**Response (Server-Sent Events)**

```
event: log.info
data: {"timestamp": "2024-05-12T13:45:00Z", "message": "Resource loaded", "logger": "resource-manager"}

event: log.warning
data: {"timestamp": "2024-05-12T13:50:00Z", "message": "Rate limit approaching", "logger": "rate-limiter"}
```

## Error Handling

The API uses standard HTTP status codes to indicate success or failure:

- `200 OK`: Successfully completed the request
- `201 Created`: Resource successfully created
- `204 No Content`: Request completed with no response body
- `400 Bad Request`: Invalid request parameters
- `401 Unauthorized`: Missing or invalid authentication
- `403 Forbidden`: Authenticated but insufficient permissions
- `404 Not Found`: Resource not found
- `429 Too Many Requests`: Rate limit exceeded
- `500 Internal Server Error`: Server-side error

Error responses include a standardized JSON structure:

```json
{
  "status": 400,
  "code": "INVALID_PARAMETER",
  "message": "Invalid query parameter",
  "details": [
    {
      "field": "limit",
      "error": "Must be a positive integer"
    }
  ],
  "_links": {
    "documentation": {"href": "https://docs.example.com/errors/INVALID_PARAMETER"}
  }
}
```

## Versioning

The API uses a date-based versioning scheme:

- Version is specified in the base path: `/mcp/v1`
- Major version changes are reflected in the path
- Minor, backward-compatible changes are handled via content negotiation

Clients should specify the API version they're targeting in the `Accept` header:

```
Accept: application/json;version=2024-11-05
```

## Hypermedia Controls

All responses include hypermedia controls using the HAL (Hypertext Application Language) format via the `_links` property. This enables API discovery and navigation.

## Rate Limiting

Rate limits are communicated via HTTP headers:

- `X-RateLimit-Limit`: Total requests allowed in the period
- `X-RateLimit-Remaining`: Requests remaining in the period
- `X-RateLimit-Reset`: Time when the limit resets (Unix timestamp)

When rate limits are exceeded, the API returns a `429 Too Many Requests` response.

## Caching

The API leverages HTTP caching mechanisms:

- `ETag` headers for resource validation
- `Last-Modified` for time-based validation
- `Cache-Control` directives for cache behavior
- Conditional requests using `If-None-Match` and `If-Modified-Since`

## Common Query Parameters

These parameters are supported across multiple endpoints:

- `fields`: Comma-separated list of fields to include (partial responses)
- `expand`: Related resources to expand inline
- `limit`: Maximum number of items to return (pagination)
- `cursor`: Pagination cursor for retrieving the next page
