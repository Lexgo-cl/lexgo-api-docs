# API Overview

Lexgo exposes two public HTTP surfaces. This overview covers both; most pages
under API Reference document the LexgoSign `/v1` envelope API.

| Surface | Base path | Auth |
|---------|-----------|------|
| **LexgoSign API** | `/v1` | API key in `Authorization` |
| **Client API** | `/client` | OAuth Bearer (`client` scope) + `X-Enterprise-Id` |

!!! note "Gateway path"
    Externally, `api.lexgo.cl` strips the `/api` prefix. Call
    `https://api.lexgo.cl/v1/...` or `https://api.lexgo.cl/client/...`.

## Base URL

### LexgoSign API

```
https://api.lexgo.cl/v1
```

### Client API

```
https://api.lexgo.cl/client
```

!!! note "Gateway path"
    Externally, the `/api` prefix is stripped: call `https://api.lexgo.cl/client/…`

!!! note "Sandbox vs Live"
    Sandbox is an API **key stage**, not a different host. Use the same
    `api.lexgo.cl` base URL with a sandbox or live key. There is no
    `sandbox-api.lexgo.cl`. See [Sandbox vs Live](../guides/sandbox.md).

## Authentication

### LexgoSign API

All LexgoSign requests require an API key in the `Authorization` header:

```
Authorization: YOUR_API_KEY
```

See [Authentication](../getting-started/authentication.md) for details.

### Client API

```
Authorization: Bearer <oauth_access_token>
X-Enterprise-Id: <enterprise_uuid>
```



## Request Format

### Content Type

Envelope creation uses multipart/form-data:

```
Content-Type: multipart/form-data
```

Other endpoints may use JSON or form-data depending on the operation.

### Request Body

Envelope requests use form-data with bracket notation:

```http
name=Contract Name
recipients[0][name]=John Doe
recipients[0][email]=john@example.com
documents[0][base64]=JVBERi0xLjQK...
documents[0][name]=contract.pdf
```

## Response Format

### Success Response

All successful responses include the requested data and a `request_id`:

```json
{
  "envelope": {...},
  "request_id": "req_abc123"
}
```

### Error Response

HTTP failures can include an error message and `request_id` (for example, 404 Not Found):

```json
{
  "error": "Envelope not found",
  "request_id": "req_def456"
}
```

Creation validation failures return HTTP 422 with the existing envelope response, including `envelope.id`, `status: ERROR`, and actionable `envelope.errors`. Fetching or updating an existing envelope can still return HTTP 200 with validation errors; inspect both HTTP status and the envelope fields:

```json
{
  "envelope": {
    "id": "envelope-id",
    "status": "ERROR",
    "errors": ["Validation error"],
    "warnings": []
  },
  "request_id": "request-id"
}
```

## HTTP Status Codes

| Code | Meaning | Description |
|------|---------|-------------|
| `200` | OK | HTTP request processed; inspect envelope `status`, `errors`, and `warnings` |
| `202` | Accepted | Send accepted asynchronously; envelope is `SENDING` |
| `401` | Unauthorized | Missing or invalid API key |
| `404` | Not Found | Resource doesn't exist |
| `405` | Method Not Allowed | Invalid operation for resource state |
| `422` | Unprocessable Entity | Validation error |
| `429` | Too Many Requests | Rate limit exceeded |
| `500` | Internal Server Error | Server error |

## Endpoints

### Envelopes

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/envelopes` | Create a new envelope |
| `GET` | `/envelopes/:id` | Get envelope details |
| `PUT` | `/envelopes/:id/send` | Send envelope to recipients |
| `PUT` | `/envelopes/:id/void` | Cancel an envelope |
| `GET` | `/envelopes/:id/evidence` | Get evidence sheet URL |

See [Envelopes API](envelopes.md) for detailed documentation.

### Validations (2FA)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/envelopes/:envelope_id/recipients/:recipient_id/validations/send_code` | Send verification code |
| `POST` | `/envelopes/:envelope_id/recipients/:recipient_id/validations/verify_code` | Verify submitted code |

See [Validations API](validations.md) for detailed documentation.

### Settings

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/settings` | Get current configuration |
| `PUT` | `/settings` | Update configuration |

See [Settings API](settings.md) for detailed documentation.

### Webhooks

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/webhooks` | List all webhooks |
| `POST` | `/webhooks` | Create a webhook |
| `GET` | `/webhooks/:id` | Get webhook details |
| `PUT` | `/webhooks/:id` | Update webhook |
| `DELETE` | `/webhooks/:id` | Delete webhook |

See [Webhooks API](webhooks.md) for detailed documentation.


## Request IDs

Every API response includes a `request_id` field:

```json
{
  "envelope": {...},
  "request_id": "req_abc123"
}
```

Use request IDs when contacting support to help us debug issues quickly.

## Rate Limiting

Rate limits protect the API from abuse and ensure fair usage:

| Resource | Limit | Window |
|----------|-------|--------|
| Email validation codes | 5 requests | 1 hour |
| Code verification attempts | 3 attempts | Per code |
| Cooldown period | 60 seconds | Between codes |
| API requests | Based on plan | 1 hour |

### Rate Limit Headers

Responses include rate limit information:

```
X-RateLimit-Limit: 5
X-RateLimit-Remaining: 3
X-RateLimit-Reset: 1635724800
```

### Exceeded Rate Limit

When rate limited, you'll receive a 429 status:

```json
{
  "error": "Wait 45 seconds before requesting another code",
  "request_id": "req_ghi789"
}
```

## Pagination

Currently, the API does not implement pagination. All results are returned in a single response.

## Versioning

The API version is included in the URL path:

```
/v1/envelopes
     ^^
  version
```

### Version History

| Version | Status | Release Date | Notes |
|---------|--------|--------------|-------|
| v1 | Current | 2023-01-01 | Initial release |

### Deprecation Policy

When we deprecate an API version:

1. **6 months notice**: Announcement via email and documentation
2. **Parallel support**: Old and new versions work simultaneously
3. **Migration guide**: Detailed upgrade instructions provided
4. **Sunset date**: Clear end-of-life timeline

## Idempotency

GET reads the current resource state; it can change while signing proceeds.
Do not assume repeated state-changing requests return the same response:
`PUT /v1/envelopes/:id/send` accepts a valid CREATED envelope with HTTP 202,
then rejects another send once its status has changed. Updates can rebuild
an unsent envelope's documents and recipients.

Creating the same envelope twice creates two separate envelopes. After a
request timeout, reconcile the known envelope ID before retrying a mutation.
Use the [envelope lifecycle](envelopes.md#envelope-states) to decide the next action.

## HTTPS Required

All API requests must use HTTPS. Requests made over plain HTTP will fail:

```
https://api.lexgo.cl/v1/envelopes  ✅
http://api.lexgo.cl/v1/envelopes   ❌
```

## CORS

Cross-Origin Resource Sharing (CORS) is supported for browser-based applications.

Allowed origins:
- `https://app.lexgo.cl`
- Custom domains (contact support to whitelist)

## Errors

See [Error Codes](errors.md) for complete error documentation.

## Next Steps

- [Envelopes API](envelopes.md) - Create and manage envelopes
- [Validations API](validations.md) - Implement 2FA email verification
- [Webhooks API](webhooks.md) - Subscribe to real-time events
