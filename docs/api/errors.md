# Error Codes

This page documents all possible error responses from the LexgoSign API.

## Error Response Format

HTTP failures generally use the following format. Envelope validation is a separate response layer:

```json
{
  "error": "Error message description",
  "request_id": "req_abc123"
}
```

## HTTP Status Codes

### 200 OK
Creation returns HTTP 200 when the envelope is valid. Fetch/update can return
an existing envelope with `status: ERROR` and nonempty `errors`; inspect
`envelope.status`, `envelope.errors`, and `envelope.warnings` before sending.

```json
{"envelope":{"id":"envelope-id","status":"ERROR","errors":["Validation error"],"warnings":[]},"request_id":"request-id"}
```

### 202 Accepted

`PUT /v1/envelopes/:id/send` queues asynchronous processing and returns an
envelope with `status: SENDING`. It does not prove invitations were delivered
or signing completed. Follow webhooks and reconcile with
`GET /v1/envelopes/:id` for `IN_PROGRESS`, `SUCCESS`, `VOIDED`, or
`ERROR`. Do not blindly retry create/send after a timeout; inspect the
known envelope first.

### 401 Unauthorized

#### Missing Authorization Header
```json
{
  "error": "Missing Authorization Header"
}
```

#### Invalid API Key
```json
{
  "error": "Invalid Api Key (key not found)"
}
```

#### Deactivated API Key
```json
{
  "error": "Invalid Api Key (key deactivated)"
}
```

### 404 Not Found

#### Envelope Not Found
```json
{
  "error": "Envelope not found",
  "request_id": "req_abc123"
}
```

#### Recipient Not Found
```json
{
  "error": "Recipient not found",
  "request_id": "req_def456"
}
```

### 405 Method Not Allowed

#### Invalid Envelope State
```json
{
  "error": "Envelope not in CREATED status",
  "request_id": "req_ghi789"
}
```

### 422 Unprocessable Entity

Creation validation failures return HTTP 422 with `envelope.id`,
`status: ERROR`, and the existing actionable `envelope.errors` array.
Keep the ID and repair the envelope with `PUT /v1/envelopes/:id` before sending.

#### Envelope Creation Validation
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

#### Verification Code
```json
{
  "error": "Invalid verification code",
  "request_id": "req_jkl012"
}
```

### 429 Too Many Requests

#### Rate Limit Exceeded
```json
{
  "error": "Wait 45 seconds before requesting another code",
  "request_id": "req_mno345"
}
```

#### Max Attempts Exceeded
```json
{
  "error": "Maximum send attempts (5) exceeded within the last hour",
  "request_id": "req_pqr678"
}
```

### 500 Internal Server Error

```json
{
  "error": "Internal server error",
  "request_id": "req_stu901"
}
```

## Error Handling Best Practices

1. Check HTTP status and the envelope lifecycle, errors, and warnings
2. Log request_id for support tickets
3. Implement exponential backoff for rate limits
4. Show user-friendly error messages
5. Monitor error rates
