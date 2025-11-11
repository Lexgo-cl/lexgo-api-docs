# Error Codes

This page documents all possible error responses from the LexgoSign API.

## Error Response Format

All error responses follow a consistent format:

```json
{
  "error": "Error message description",
  "request_id": "req_abc123"
}
```

## HTTP Status Codes

### 200 OK
Request succeeded. Resource returned in response body.

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

#### Validation Errors
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

1. Always check HTTP status code
2. Log request_id for support tickets
3. Implement exponential backoff for rate limits
4. Show user-friendly error messages
5. Monitor error rates
