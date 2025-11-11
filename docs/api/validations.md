# Validations API (2FA)

The Validations API provides email-based two-factor authentication (2FA) for envelope recipients. Recipients must verify their email address with a 6-digit code before accessing documents.

!!! info "Opt-In Feature"
    Email validation is disabled by default. Enable it per envelope in the settings.

## Overview

Email validation adds a security layer by requiring recipients to:

1. Request a 6-digit verification code sent to their email
2. Enter the code to verify email ownership
3. Access the document only after successful verification

## Security Features

- **SHA256 Hashing**: Codes stored as cryptographic hashes, never plain text
- **Rate Limiting**: 5 codes per hour, 60-second cooldown between requests
- **Attempt Limiting**: Maximum 3 verification attempts per code
- **30-Minute Expiration**: Codes automatically expire
- **Timing Attack Prevention**: Constant-time code comparison
- **Evidence Tracking**: Complete audit trail for compliance

---

## Send Verification Code

Send a 6-digit verification code to the recipient's email address.

### Endpoint

```
POST /api/v1/envelopes/:envelope_id/recipients/:recipient_id/validations/send_code
```

### Request

```bash
curl -X POST https://api.lexgo.cl/api/v1/envelopes/550e8400-e29b-41d4-a716-446655440000/recipients/123e4567-e89b-12d3-a456-426614174000/validations/send_code \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json"
```

### Success Response

**Status**: `200 OK`

```json
{
  "success": true,
  "expires_at": "2025-11-01T11:00:00Z",
  "request_id": "req_abc123"
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | boolean | Always `true` for successful requests |
| `expires_at` | string (ISO 8601) | Code expiration timestamp (30 minutes from now) |
| `request_id` | string | Unique request identifier |

### Error Responses

#### Rate Limited (Cooldown)

**Status**: `429 Too Many Requests`

```json
{
  "error": "Wait 45 seconds before requesting another code",
  "request_id": "req_def456"
}
```

Occurs when requesting a code within the 60-second cooldown period.

#### Max Attempts Exceeded

**Status**: `429 Too Many Requests`

```json
{
  "error": "Maximum send attempts (5) exceeded within the last hour",
  "request_id": "req_ghi789"
}
```

Occurs when 5 codes have been sent within the last hour.

#### Envelope Not Found

**Status**: `404 Not Found`

```json
{
  "error": "Envelope not found",
  "request_id": "req_jkl012"
}
```

#### Recipient Not Found

**Status**: `404 Not Found`

```json
{
  "error": "Recipient not found",
  "request_id": "req_mno345"
}
```

### Email Content

Recipients receive an email with:
- **Subject**: "Your verification code for {Envelope Name}"
- **Code**: 6-digit numeric code (e.g., `123456`)
- **Expiration**: 30 minutes
- **Sender**: Configurable via email templates

---

## Verify Code

Verify a submitted 6-digit code.

### Endpoint

```
POST /api/v1/envelopes/:envelope_id/recipients/:recipient_id/validations/verify_code
```

### Request

```bash
curl -X POST https://api.lexgo.cl/api/v1/envelopes/550e8400-e29b-41d4-a716-446655440000/recipients/123e4567-e89b-12d3-a456-426614174000/validations/verify_code \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"code": "123456"}'
```

### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `code` | string | Yes | 6-digit verification code |

### Success Response

**Status**: `200 OK`

```json
{
  "success": true,
  "message": "Email validated successfully",
  "request_id": "req_pqr678"
}
```

### Error Responses

#### Invalid Code

**Status**: `422 Unprocessable Entity`

```json
{
  "error": "Invalid verification code",
  "request_id": "req_stu901"
}
```

Occurs when the submitted code doesn't match the stored hash.

#### Expired Code

**Status**: `422 Unprocessable Entity`

```json
{
  "error": "Verification code has expired",
  "request_id": "req_vwx234"
}
```

Occurs when the code is older than 30 minutes.

#### Max Attempts Exceeded

**Status**: `422 Unprocessable Entity`

```json
{
  "error": "Maximum verification attempts (3) exceeded",
  "request_id": "req_yz567"
}
```

Occurs after 3 failed verification attempts. Recipient must request a new code.

#### No Pending Validation

**Status**: `404 Not Found`

```json
{
  "error": "No pending validation found",
  "request_id": "req_abc890"
}
```

Occurs when no code has been sent or all codes have been used/expired.

---

## Complete Flow Example

### Python

```python
import requests
import time

BASE_URL = 'https://api.lexgo.cl/api/v1'
ENVELOPE_ID = '550e8400-e29b-41d4-a716-446655440000'
RECIPIENT_ID = '123e4567-e89b-12d3-a456-426614174000'

headers = {
    'Authorization': 'YOUR_API_KEY',
    'Content-Type': 'application/json'
}

# Step 1: Send verification code
response = requests.post(
    f'{BASE_URL}/envelopes/{ENVELOPE_ID}/recipients/{RECIPIENT_ID}/validations/send_code',
    headers=headers
)

if response.status_code == 200:
    data = response.json()
    print(f"Code sent! Expires at: {data['expires_at']}")
elif response.status_code == 429:
    print(f"Rate limited: {response.json()['error']}")
    exit(1)

# Step 2: Wait for user to receive email and enter code
code = input("Enter the 6-digit code from email: ")

# Step 3: Verify the code
response = requests.post(
    f'{BASE_URL}/envelopes/{ENVELOPE_ID}/recipients/{RECIPIENT_ID}/validations/verify_code',
    headers=headers,
    json={'code': code}
)

if response.status_code == 200:
    print("✓ Email validated successfully!")
else:
    error = response.json()['error']
    print(f"✗ Verification failed: {error}")
```

### Node.js

```javascript
const fetch = require('node-fetch');

const BASE_URL = 'https://api.lexgo.cl/api/v1';
const ENVELOPE_ID = '550e8400-e29b-41d4-a716-446655440000';
const RECIPIENT_ID = '123e4567-e89b-12d3-a456-426614174000';

const headers = {
  'Authorization': 'YOUR_API_KEY',
  'Content-Type': 'application/json'
};

// Step 1: Send verification code
const sendResponse = await fetch(
  `${BASE_URL}/envelopes/${ENVELOPE_ID}/recipients/${RECIPIENT_ID}/validations/send_code`,
  { method: 'POST', headers }
);

if (sendResponse.status === 200) {
  const data = await sendResponse.json();
  console.log(`Code sent! Expires at: ${data.expires_at}`);
} else if (sendResponse.status === 429) {
  const error = await sendResponse.json();
  console.log(`Rate limited: ${error.error}`);
  process.exit(1);
}

// Step 2: Get code from user
const code = await getUserInput("Enter the 6-digit code from email: ");

// Step 3: Verify the code
const verifyResponse = await fetch(
  `${BASE_URL}/envelopes/${ENVELOPE_ID}/recipients/${RECIPIENT_ID}/validations/verify_code`,
  {
    method: 'POST',
    headers,
    body: JSON.stringify({ code })
  }
);

if (verifyResponse.status === 200) {
  console.log('✓ Email validated successfully!');
} else {
  const error = await verifyResponse.json();
  console.log(`✗ Verification failed: ${error.error}`);
}
```

---

## Enable Email Validation

To enable email validation for an envelope, configure it in the envelope settings:

```json
{
  "envelope": {
    "name": "Contract",
    "settings": {
      "validations": {
        "email": {
          "enabled": "TRUE",
          "order": 0
        }
      }
    },
    "envelope_signers": [...]
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `enabled` | string | `"TRUE"` to enable, `"FALSE"` to disable |
| `order` | integer | Validation order (0 = before document access) |

!!! tip "When to Use"
    Enable email validation for sensitive documents, legal contracts, or when identity verification is critical.

---

## Webhook Events

Email validation triggers several webhook events:

| Event | Description | Trigger |
|-------|-------------|---------|
| `recipient.validation_code_generated` | Code generated | After code creation |
| `recipient.validation_code_sent` | Email sent | After email delivery |
| `recipient.validation_code_success` | Code verified | After successful verification |
| `recipient.validation_code_failed` | Invalid code | After failed verification |
| `recipient.validation_code_expired` | Code expired | When code expires |
| `recipient.validation_max_attempts` | Max attempts | After 3 failed attempts |

See [Webhook Integration](../guides/webhook-integration.md) for details.

---

## Rate Limits

| Limit | Value | Window |
|-------|-------|--------|
| Send code requests | 5 | 1 hour |
| Cooldown between sends | 60 seconds | Per request |
| Verification attempts | 3 | Per code |
| Code expiration | 30 minutes | Per code |

---

## Evidence Tracking

All validation events are automatically tracked in the evidence sheet:

- Code generation timestamp
- Email delivery confirmation (AWS SES Message ID)
- Verification attempts (success and failures)
- Expiration events
- IP addresses and timestamps

Evidence appears in the envelope timeline and evidence sheet PDF.

---

## Best Practices

### User Experience

- **Clear instructions**: Inform users they'll receive a code via email
- **Expiration warning**: Show countdown timer (30 minutes)
- **Resend option**: Allow users to request new codes
- **Error handling**: Display clear error messages for failures

### Security

- **Don't log codes**: Never log verification codes in plaintext
- **Monitor attempts**: Track failed verification patterns
- **Alert on abuse**: Set up alerts for excessive failures
- **Rate limit client-side**: Prevent unnecessary API calls

### Integration

- **Validate envelope status**: Ensure envelope is in correct state
- **Handle rate limits**: Implement exponential backoff
- **Track expiration**: Show users time remaining
- **Test thoroughly**: Test all error scenarios

---

## Common Issues

### Code Not Received

1. **Check spam folder**: Emails may be filtered
2. **Verify email address**: Ensure recipient email is correct
3. **AWS SES limits**: Check SES sending quotas
4. **Email delivery webhooks**: Monitor delivery status

### Rate Limiting

1. **Wait for cooldown**: 60 seconds between requests
2. **Don't retry immediately**: Implement exponential backoff
3. **Show timer**: Display countdown to users
4. **Request new code**: After expiration, generate new code

### Verification Failures

1. **Check code accuracy**: Ensure no typos (numeric only)
2. **Check expiration**: Codes expire after 30 minutes
3. **Attempt limit**: Max 3 attempts per code
4. **Request new code**: After failures, send new code

---

## Next Steps

- [Email Validation Guide](../guides/email-validation.md) - Complete implementation guide
- [Custom Email Templates](../guides/email-templates.md) - Customize validation emails
- [Webhook Integration](../guides/webhook-integration.md) - Track validation events
- [Evidence Sheet](../guides/evidence-sheet.md) - Compliance and audit trails
