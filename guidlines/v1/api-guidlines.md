# REST API Design — Best Practices Guidelines

> A comprehensive reference for designing clean, consistent, and developer-friendly REST APIs.

---

## Table of Contents

1. [What Makes a Good API?](#1-what-makes-a-good-api)
2. [URL & Resource Design](#2-url--resource-design)
3. [HTTP Methods & CRUD](#3-http-methods--crud)
4. [HTTP Status Codes](#4-http-status-codes)
5. [Request Design](#5-request-design)
6. [Response Design](#6-response-design)
7. [Error Handling](#7-error-handling)
8. [Versioning](#8-versioning)
9. [Authentication & Authorization](#9-authentication--authorization)
10. [Pagination, Filtering & Sorting](#10-pagination-filtering--sorting)
11. [Security Best Practices](#11-security-best-practices)
12. [Performance Best Practices](#12-performance-best-practices)
13. [Documentation](#13-documentation)
14. [Dos and Don'ts — Quick Reference](#14-dos-and-donts--quick-reference)

---

## 1. What Makes a Good API?

A well-designed API should have the following core characteristics:

| Property | Description |
|---|---|
| **Easy to read** | Resources and operations are intuitive and quickly memorized by developers |
| **Hard to misuse** | Clear structure and feedback prevent incorrect usage |
| **Complete & concise** | Exposes all data needed for full-fledged applications without unnecessary complexity |
| **Consistent** | Naming conventions, response shapes, and error formats are uniform throughout |
| **Predictable** | Developers can guess how an undocumented endpoint works based on existing patterns |

---

## 2. URL & Resource Design

### Use Nouns, Not Verbs

URLs should represent **resources (things)**, not **actions (verbs)**. The HTTP method conveys the action.

```
✅  GET  /users
✅  POST /users
❌  GET  /getUsers
❌  POST /createUser
```

### Use Plural Nouns for Collections

```
✅  /users          → collection of users
✅  /users/{id}     → a specific user
✅  /photos         → collection of photos
❌  /user
❌  /photo
```

### Use Hierarchical Structure for Relationships

Nest related resources under their parent, but avoid going more than 2–3 levels deep.

```
✅  GET /users/{userId}/photos          → all photos by a user
✅  GET /users/{userId}/photos/{photoId} → a specific photo by a user
❌  GET /users/{userId}/photos/{photoId}/comments/{commentId}/likes  ← too deep
```

### Keep URLs Lowercase and Use Hyphens

```
✅  /photo-albums
✅  /user-profiles
❌  /PhotoAlbums
❌  /user_profiles
```

---

## 3. HTTP Methods & CRUD

Use HTTP methods semantically and consistently:

| Method | Action | Example |
|---|---|---|
| `GET` | Retrieve a resource or collection | `GET /users` |
| `POST` | Create a new resource | `POST /users` |
| `PUT` | Replace an existing resource entirely | `PUT /users/{id}` |
| `PATCH` | Partially update an existing resource | `PATCH /users/{id}` |
| `DELETE` | Delete a resource | `DELETE /users/{id}` |

### PUT vs PATCH

- **PUT** — replaces the entire resource. Missing fields revert to defaults.
- **PATCH** — updates only the fields provided in the request body.

```json
// PATCH /users/42  — only update the email
{
  "email": "new@example.com"
}
```

### Idempotency

| Method | Idempotent? | Safe? |
|---|---|---|
| GET | ✅ Yes | ✅ Yes |
| PUT | ✅ Yes | ❌ No |
| PATCH | ❌ No | ❌ No |
| POST | ❌ No | ❌ No |
| DELETE | ✅ Yes | ❌ No |

---

## 4. HTTP Status Codes

Always return meaningful HTTP status codes. Never return `200 OK` for an error.

### 2xx — Success

| Code | Meaning | When to Use |
|---|---|---|
| `200 OK` | Request succeeded | Successful GET, PUT, PATCH |
| `201 Created` | Resource created | Successful POST |
| `204 No Content` | Success, no body returned | Successful DELETE |

### 4xx — Client Errors

| Code | Meaning | When to Use |
|---|---|---|
| `400 Bad Request` | Malformed request or invalid data | Validation failures |
| `401 Unauthorized` | Authentication required | Missing or invalid token |
| `403 Forbidden` | Authenticated but not authorized | Insufficient permissions |
| `404 Not Found` | Resource doesn't exist | Wrong ID or endpoint |
| `409 Conflict` | State conflict | Duplicate entry |
| `422 Unprocessable Entity` | Validation error on well-formed request | Business rule violations |
| `429 Too Many Requests` | Rate limit exceeded | Throttling |

### 5xx — Server Errors

| Code | Meaning | When to Use |
|---|---|---|
| `500 Internal Server Error` | Unexpected server failure | Unhandled exceptions |
| `502 Bad Gateway` | Upstream service failure | Gateway/proxy issues |
| `503 Service Unavailable` | Server overloaded or down | Maintenance windows |

---

## 5. Request Design

### Use Query Parameters for Filtering, Searching & Limiting

```
GET /photos?location=boston&hashtag=winter&limit=10
GET /users?role=admin&status=active
GET /products?min_price=10&max_price=100
```

### Use Path Parameters for Specific Resources

```
GET /users/{userId}
GET /orders/{orderId}/items/{itemId}
```

### Use Request Body for Creating/Updating Resources

```json
POST /users
Content-Type: application/json

{
  "username": "johndoe",
  "email": "john@example.com",
  "role": "editor"
}
```

### Content Negotiation

Always include and respect `Content-Type` and `Accept` headers:

```
Content-Type: application/json
Accept: application/json
```

---

## 6. Response Design

### Wrap Responses in a Consistent Envelope

Maintain a predictable response structure across all endpoints:

```json
// Success response
{
  "status": "success",
  "data": {
    "id": 1,
    "username": "johndoe",
    "email": "john@example.com",
    "created_at": "2024-01-15T10:30:00Z"
  }
}

// Collection response
{
  "status": "success",
  "data": [...],
  "meta": {
    "total": 150,
    "page": 1,
    "per_page": 10,
    "total_pages": 15
  }
}
```

### Use camelCase for JSON Fields

```json
✅  { "firstName": "John", "createdAt": "2024-01-01" }
❌  { "first_name": "John", "created_at": "2024-01-01" }
```

> Note: Be consistent — pick one convention (camelCase or snake_case) and stick to it.

### Use ISO 8601 for Dates & Times

```json
✅  "createdAt": "2024-01-15T10:30:00Z"
❌  "createdAt": "01/15/2024"
❌  "createdAt": 1705312200
```

### Only Return What's Needed

Avoid over-fetching. Don't expose sensitive or irrelevant fields in responses:

```json
❌  { "id": 1, "username": "john", "passwordHash": "...", "internalFlag": true }
✅  { "id": 1, "username": "john", "email": "john@example.com" }
```

---

## 7. Error Handling

### Provide Descriptive, Actionable Error Messages

```json
// ❌ Vague error
{
  "error": "Bad Request"
}

// ✅ Descriptive error
{
  "status": "error",
  "code": 400,
  "message": "Validation failed",
  "errors": [
    {
      "field": "email",
      "issue": "Must be a valid email address"
    },
    {
      "field": "password",
      "issue": "Must be at least 8 characters"
    }
  ],
  "docs": "https://api.example.com/docs/errors#validation"
}
```

### Standard Error Response Shape

Every error response should consistently include:

| Field | Description |
|---|---|
| `status` | `"error"` |
| `code` | HTTP status code |
| `message` | Human-readable summary |
| `errors` | (Optional) Array of field-level issues |
| `docs` | (Optional) Link to relevant documentation |

### Never Expose Internal Details

```json
❌  { "error": "NullPointerException at UserService.java:142" }
✅  { "error": "An unexpected error occurred. Reference ID: abc-123" }
```

---

## 8. Versioning

Always version your API to allow for non-breaking evolution.

### URL Versioning (Most Common)

```
https://api.example.com/v1/users
https://api.example.com/v2/users
```

### Header Versioning

```
GET /users
Accept: application/vnd.example.v2+json
```

### Best Practices

- Start with `v1` from day one — even before you need `v2`
- Never make breaking changes within the same version
- Maintain older versions for a reasonable deprecation window (e.g., 6–12 months)
- Clearly document deprecation timelines and migration paths

---

## 9. Authentication & Authorization

### Use Industry-Standard Auth Mechanisms

| Method | Use Case |
|---|---|
| **API Keys** | Simple server-to-server integrations |
| **OAuth 2.0** | Third-party access delegation |
| **JWT (Bearer Tokens)** | Stateless user authentication |
| **OpenID Connect** | Identity layer on top of OAuth 2.0 |

### Sending Auth Credentials

Always use the `Authorization` header — never query parameters or the URL:

```
✅  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5c...
❌  GET /users?token=eyJhbGciOiJIUzI1NiIsInR5c...
```

### Principle of Least Privilege

Grant only the permissions a client needs. Use scopes to restrict access:

```
scope: read:users write:photos
```

---

## 10. Pagination, Filtering & Sorting

### Pagination

Always paginate large collections. Never return unbounded lists.

```
GET /users?page=2&per_page=25
GET /users?offset=50&limit=25
GET /users?cursor=eyJpZCI6NTB9   ← cursor-based (preferred for large datasets)
```

Include pagination metadata in the response:

```json
{
  "data": [...],
  "meta": {
    "total": 500,
    "page": 2,
    "per_page": 25,
    "total_pages": 20,
    "next": "/users?page=3&per_page=25",
    "prev": "/users?page=1&per_page=25"
  }
}
```

### Filtering

```
GET /products?category=electronics&in_stock=true
GET /orders?status=pending&created_after=2024-01-01
```

### Sorting

```
GET /users?sort=created_at&order=desc
GET /products?sort=price&order=asc
```

---

## 11. Security Best Practices

- **Always use HTTPS** — never serve APIs over plain HTTP
- **Validate all inputs** — never trust client data; validate on the server side
- **Implement rate limiting** — protect against abuse and DDoS (`429 Too Many Requests`)
- **Use CORS appropriately** — whitelist trusted origins, don't use `*` in production
- **Sanitize outputs** — prevent injection attacks by encoding data before returning it
- **Rotate secrets regularly** — API keys, tokens, and signing secrets should expire
- **Log and monitor** — track unusual patterns and alert on anomalies
- **Never log sensitive data** — passwords, tokens, and PII should never appear in logs

---

## 12. Performance Best Practices

### Caching

Use HTTP cache headers to reduce redundant requests:

```
Cache-Control: max-age=3600, public
ETag: "abc123"
Last-Modified: Tue, 15 Jan 2024 10:30:00 GMT
```

Support conditional requests:

```
GET /users/42
If-None-Match: "abc123"
→ 304 Not Modified  (if unchanged)
```

### Compression

Enable `gzip` or `br` compression for response bodies:

```
Accept-Encoding: gzip, deflate, br
Content-Encoding: gzip
```

### Field Selection (Sparse Fieldsets)

Let clients request only the fields they need:

```
GET /users?fields=id,username,email
```

### Asynchronous Processing

For long-running operations, return `202 Accepted` and provide a status endpoint:

```json
// POST /reports  →  202 Accepted
{
  "status": "processing",
  "jobId": "job_abc123",
  "statusUrl": "/jobs/job_abc123"
}
```

---

## 13. Documentation

Good documentation is as important as good code.

### Must-Have Documentation Elements

- **Overview** — what the API does and who it's for
- **Authentication guide** — how to obtain and use credentials
- **Base URL & versioning** — environment-specific URLs
- **Endpoint reference** — method, path, parameters, request body, responses
- **Code examples** — real request/response samples in multiple languages
- **Error reference** — all possible error codes and their meanings
- **Changelog** — what changed and when, especially breaking changes

### Use OpenAPI / Swagger Specification

Define your API using the [OpenAPI Specification (OAS)](https://swagger.io/specification/) to:

- Auto-generate interactive documentation (Swagger UI)
- Enable client SDK generation
- Support contract testing
- Ensure design consistency

```yaml
# Example OpenAPI snippet
paths:
  /users/{userId}:
    get:
      summary: Get a user by ID
      parameters:
        - name: userId
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: User found
        '404':
          description: User not found
```

---

## 14. Dos and Don'ts — Quick Reference

### ✅ DO

- Use nouns for resource names and keep them plural
- Use HTTP methods to convey actions (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`)
- Return appropriate HTTP status codes
- Version your API from the start (`/v1/`)
- Validate and sanitize all inputs server-side
- Paginate all collection responses
- Use HTTPS everywhere
- Provide clear, actionable error messages
- Document with OpenAPI/Swagger
- Use ISO 8601 for all dates and times
- Implement rate limiting

### ❌ DON'T

- Use verbs in URLs (`/getUsers`, `/deleteOrder`)
- Return `200 OK` for errors
- Expose sensitive data (passwords, tokens, internal stack traces)
- Break backwards compatibility within the same API version
- Use query parameters for authentication tokens
- Return unbounded collections without pagination
- Ignore CORS — misconfigured CORS is a security risk
- Use inconsistent naming conventions across endpoints
- Skip documentation — undocumented APIs are unusable APIs
- Use custom error formats per endpoint — keep errors consistent

---

## References

- [Swagger — Best Practices in API Design](https://swagger.io/resources/articles/best-practices-in-api-design/)
- [Mastering REST API Design — Medium](https://medium.com/@syedabdullahrahman/mastering-rest-api-design-essential-best-practices-dos-and-don-ts-for-2024-dd41a2c59133)
- [OpenAPI Specification](https://swagger.io/specification/)
- [HTTP Status Codes Reference](https://www.restapitutorial.com/httpstatuscodes.html)
- [RFC 7231 — HTTP/1.1 Semantics](https://tools.ietf.org/html/rfc7231)

---

*Last updated: February 2026*
