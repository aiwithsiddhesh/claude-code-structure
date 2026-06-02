---
paths:
  - "src/api/**"
  - "**/routes/**"
  - "**/controllers/**"
  - "**/handlers/**"
  - "api/**"
  - "output/**/src/api/**"
  - "output/**/api/**"
  - "output/**/routes/**"
  - "output/**/controllers/**"
  - "output/**/handlers/**"
---

# API Patterns

Loaded when working with API routes, controllers, and handlers.

## URL Conventions

- **Plural nouns** for resource collections: `/users`, `/orders`, `/products`
- **Lowercase, hyphenated**: `/user-profiles`, not `/userProfiles` or `/user_profiles`
- **No verbs in URLs** — the HTTP method is the verb:
  - `GET /orders` — list orders
  - `POST /orders` — create order
  - `GET /orders/:id` — get order
  - `PUT /orders/:id` — replace order
  - `PATCH /orders/:id` — partial update
  - `DELETE /orders/:id` — delete order
- **Nested resources** for owned sub-resources: `GET /users/:userId/orders`
- **Actions that don't fit REST** → use a POST with a descriptive noun: `POST /orders/:id/cancellation`
- **Version prefix on all routes**: `/api/v1/users`

## Standard Response Envelope

All API responses use a consistent envelope:

**Success**:
```json
{
  "data": { ... },        // the resource or array
  "meta": {               // optional, for paginated lists
    "total": 100,
    "page": 1,
    "pageSize": 20
  }
}
```

**Error**:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",     // machine-readable, UPPER_SNAKE_CASE
    "message": "Email is required", // human-readable, for display
    "details": [                    // optional, field-level errors
      { "field": "email", "message": "Email is required" }
    ]
  }
}
```

- Never return a bare object or array at the top level — always wrap in `{ data: ... }`.
- Never expose stack traces, file paths, or internal IDs in error responses.

## HTTP Status Codes

| Situation | Code |
|---|---|
| Success, resource returned | 200 OK |
| Resource created | 201 Created |
| Success, no body (e.g., DELETE) | 204 No Content |
| Bad request / validation error | 400 Bad Request |
| Missing or invalid auth token | 401 Unauthorized |
| Authenticated but not permitted | 403 Forbidden |
| Resource not found | 404 Not Found |
| Conflict (duplicate, version) | 409 Conflict |
| Input unprocessable (semantics) | 422 Unprocessable Entity |
| Rate limit exceeded | 429 Too Many Requests |
| Unexpected server error | 500 Internal Server Error |

- Never return 200 for an error. Never return 500 for a validation error.

## Pagination

All list endpoints must support pagination. Default:
```
GET /api/v1/users?page=1&pageSize=20
```

Response includes `meta.total`, `meta.page`, `meta.pageSize`.

- Default page size: 20. Max page size: 100. Requests above max return 400.
- Cursor-based pagination for high-volume or real-time data: `?cursor=xyz&limit=20`

## Filtering, Sorting, Search

```
GET /api/v1/users?status=active&sort=createdAt&order=desc&search=john
```

- Filter params: exact field names (`status`, `role`)
- Sort: `sort={field}&order=asc|desc`
- Search: `search={term}` (full-text across documented fields)
- Validate all filter/sort params — reject unknown fields with 400

## Request Validation

Every endpoint validates its input before touching the database:
1. Path params (`/users/:id`) — verify existence early, return 404 if missing
2. Query params — validate types, ranges, allowed values
3. Request body — validate with Zod (TS) or Pydantic (Python) schema before processing

Validation failures always return `400 Bad Request` with `details` array per field.

## Versioning

- Version in the URL: `/api/v1/`, `/api/v2/`
- A new version is required when: removing a field, changing a field type, changing a URL, or breaking a contract
- Old versions must remain functional for at least one major release cycle
- Document deprecated endpoints with a `Deprecation` response header

## Authentication on Routes

Every route must explicitly be:
- **Public** (documented as intentionally unauthenticated)
- **Authenticated** (requires valid token — middleware applied)
- **Authorized** (requires specific role/permission — checked in handler)

No route is implicitly protected. Make it explicit in code and documentation.
