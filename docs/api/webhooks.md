# Webhooks API

Webhooks enable real-time notifications when events occur in LexgoSign.

## Overview

Subscribe to events and receive HTTP POST requests to your server when:

- Envelopes change status
- Recipients sign documents
- Emails are delivered or opened
- Validation codes are verified

## Webhook Events

### Envelope Events

| Event | Description |
|-------|-------------|
| `envelope.in_progress` | Envelope sent to recipients |
| `envelope.success` | All recipients signed |
| `envelope.voided` | Envelope cancelled |

### Recipient Events

| Event | Description |
|-------|-------------|
| `recipient.email_sent` | Invitation email sent |
| `recipient.email_received` | Email delivered (via AWS SES) |
| `recipient.email_opened` | Recipient opened email |
| `recipient.email_clicked` | Recipient clicked link |
| `recipient.email_bounced` | Email bounced |
| `recipient.link_generated` | Unique access link created for recipient |
| `recipient.in_progress` | Recipient can now sign |
| `recipient.signed_all` | Recipient completed signing |
| `recipient.rejected` | Recipient declined |

### Validation Events (2FA)

| Event | Description |
|-------|-------------|
| `recipient.validation_code_generated` | Verification code created |
| `recipient.validation_code_sent` | Code email sent |
| `recipient.validation_code_success` | Code verified successfully |
| `recipient.validation_code_failed` | Invalid code submitted |
| `recipient.validation_code_expired` | Code expired |
| `recipient.validation_max_attempts` | Too many failed attempts |

## Create Webhook

### Endpoint

```
POST /api/v1/webhooks
```

### Request

```bash
curl -X POST https://api.lexgo.cl/api/v1/webhooks \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-app.com/webhooks/lexgo",
    "subscriptions": "envelope.success,recipient.signed_all,recipient.validation_code_success",
    "auth_type": "header",
    "auth": "Bearer your_webhook_secret"
  }'
```

### Response

```json
{
  "success": true,
  "webhook": {
    "id": "webhook_123",
    "url": "https://your-app.com/webhooks/lexgo",
    "subscriptions": "envelope.success,recipient.signed_all,recipient.validation_code_success",
    "auth_type": "HEADER",
    "created_at": "2025-11-01T10:00:00Z"
  },
  "request_id": "req_abc123"
}
```

## Webhook Payload

When an event occurs, LexgoSign sends a POST request:

```json
{
  "id": "evt_abc123",
  "event_type": "recipient.validation_code_success",
  "data": {
    "id": "recipient_123",
    "name": "John Doe",
    "email": "john.doe@example.com",
    "status": "EMAIL_VALIDATED",
    "envelope": {
      "id": "envelope_456",
      "name": "Employment Contract",
      "status": "IN_PROGRESS"
    }
  }
}
```

## List Webhooks

```bash
curl https://api.lexgo.cl/api/v1/webhooks \
  -H "Authorization: YOUR_API_KEY"
```

## Update Webhook

```bash
curl -X PUT https://api.lexgo.cl/api/v1/webhooks/{webhook_id} \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subscriptions": "envelope.success,envelope.voided"
  }'
```

## Delete Webhook

```bash
curl -X DELETE https://api.lexgo.cl/api/v1/webhooks/{webhook_id} \
  -H "Authorization: YOUR_API_KEY"
```

## Retry Logic

Failed webhook deliveries are retried:

- **3 retry attempts**
- **5-second delay** between attempts
- **Async delivery** via Sidekiq

## Best Practices

1. **Verify signatures**: Validate webhook authenticity
2. **Respond quickly**: Return 200 status immediately
3. **Process async**: Handle payload in background job
4. **Log events**: Track all webhook deliveries
5. **Handle duplicates**: Implement idempotency

## Next Steps

- [Webhook Integration Guide](../guides/webhook-integration.md)
- [Example: Webhook Subscriptions](../examples/webhook-subscriptions.md)
