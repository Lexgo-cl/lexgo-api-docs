# Webhooks API

Webhooks enable real-time notifications when events occur in LexgoSign.

## Overview

Subscribe to events and receive HTTP POST requests to your server when:

- Envelopes change status
- Files are uploaded via presigned URLs
- Evidence sheets are generated
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
| `envelope.evidence_generated` | Evidence sheet PDF generated and available |
| `envelope.file_uploaded` | File uploaded via presigned URL and ready |

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

```http
POST /api/v1/webhooks
```

### Request

```bash
curl -X POST https://api.lexgo.cl/api/v1/webhooks \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-app.com/webhooks/lexgo",
    "subscriptions": "envelope.success,envelope.evidence_generated,recipient.signed_all",
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
    "subscriptions": "envelope.success,envelope.evidence_generated,recipient.signed_all",
    "auth_type": "HEADER",
    "created_at": "2025-11-01T10:00:00Z"
  },
  "request_id": "req_abc123"
}
```

## Webhook Payload

When an event occurs, LexgoSign sends a POST request:

### File Upload Event Example

When a presigned upload completes processing:

```json
{
  "id": "event_abc123",
  "event_type": "envelope.file_uploaded",
  "data": {
    "envelope_id": "51f8ad72-0b51-4bf4-ab7f-6575e4f68ad5",
    "presigned_upload_id": "upload_xyz789",
    "file_id": "b66efe2d-a3c8-43d6-80d2-1f4e71fa0216",
    "order": 0,
    "file_name": "contract.pdf",
    "custom_key": "main_doc",
    "status": "COMPLETED",
    "uploaded_at": "2025-11-20T19:20:15-03:00",
    "processed_at": "2025-11-20T19:20:18-03:00"
  }
}
```

Use this event to know when presigned uploads are ready. See [Presigned Uploads API](presigned-uploads.md).

### Recipient Event Example

```json
{
  "id": "cbbda446-ac8a-4102-b992-b1032db17cf0",
  "event_type": "recipient.in_progress",
  "data": {
    "id": "45d6e9bf-61d6-423c-b8f0-68fdc71429ab",
    "custom_key": "0",
    "order": 0,
    "status": "IN_PROGRESS",
    "created_at": "2025-11-18T15:44:11.804-03:00",
    "updated_at": "2025-11-18T15:59:09.042-03:00",
    "name": "Jane Smith",
    "name_label": "Name:",
    "email": "recipient@company.com",
    "email_label": "Email:"
  }
}
```

### Envelope Event Example

```json
{
  "id": "d7bd4315-5838-4a9c-b5c1-e96942825078",
  "event_type": "envelope.evidence_generated",
  "data": {
    "id": "51f8ad72-0b51-4bf4-ab7f-6575e4f68ad5",
    "name": "employment_agreement_2025",
    "status": "SUCCESS",
    "errors": [],
    "warnings": [],
    "created_at": "2025-11-18T15:44:11.534-03:00",
    "updated_at": "2025-11-18T16:03:07.120-03:00",
    "evidence_document_url": "https://lexgo-secure.s3.us-east-2.amazonaws.com/evidence-sheets/2025-11-18-evidence.pdf",
    "evidence_generated_at": "2025-11-18T16:03:08.456-03:00",
    "documents": [
      {
        "id": "b66efe2d-a3c8-43d6-80d2-1f4e71fa0216",
        "custom_key": "0",
        "file_type": "SIGNABLE",
        "order": 0,
        "status": "SIGNED",
        "created_at": "2025-11-18T15:44:12.952-03:00",
        "updated_at": "2025-11-18T16:03:07.112-03:00",
        "file_name": "employment_agreement_2025.pdf",
        "content_type": "application/pdf",
        "file_size": 1023380,
        "page_count": 1,
        "url": "https://lexgo-secure.s3.us-east-2.amazonaws.com/pki/files/data/469/c50/d8-/original/document.pdf?X-Amz-Algorithm=..."
      }
    ],
    "recipients": [
      {
        "id": "45d6e9bf-61d6-423c-b8f0-68fdc71429ab",
        "custom_key": "0",
        "order": 0,
        "status": "SIGNED_ALL",
        "created_at": "2025-11-18T15:44:11.804-03:00",
        "updated_at": "2025-11-18T16:02:18.393-03:00",
        "name": "Jane Smith",
        "name_label": "Name:",
        "email": "jane.smith@company.com",
        "email_label": "Email:"
      },
      {
        "id": "e2616688-a514-455e-a0d1-92ad8b544825",
        "custom_key": "1",
        "order": 1,
        "status": "SIGNED_ALL",
        "created_at": "2025-11-18T15:44:11.833-03:00",
        "updated_at": "2025-11-18T16:03:07.044-03:00",
        "name": "Robert Johnson",
        "name_label": "Name:",
        "email": "robert.johnson@company.com",
        "email_label": "Email:"
      }
    ],
    "placements": [
      {
        "id": "a24bf7b1-d9b2-49d2-8231-68d899445a63",
        "custom_key": "4",
        "placement_type": "SIGNATURE",
        "created_at": "2025-11-18T15:44:11.944-03:00",
        "updated_at": "2025-11-18T15:44:11.944-03:00",
        "recipient_id": "45d6e9bf-61d6-423c-b8f0-68fdc71429ab",
        "recipient_key": "0",
        "document_id": "b66efe2d-a3c8-43d6-80d2-1f4e71fa0216",
        "document_key": "0",
        "order": 0,
        "coordinates": {
          "id": "Lexgo-Sign-0",
          "top": "1538.6666666667",
          "left": "2554.3333333333",
          "page": "0"
        },
        "token": null
      }
    ],
    "settings": {
      "emails": {
        "envelope_invitation": "FALSE",
        "envelope_voided": "FALSE",
        "envelope_rejected": "FALSE",
        "envelope_signed": "FALSE"
      },
      "documents": {
        "include_page_id": "TRUE",
        "custom_page_id": "CVE:",
        "signatures_per_page": 4,
        "signatures_per_row": 1
      },
      "signature_flow": {
        "language": "ES",
        "signature_order": "SEQUENTIAL",
        "signature_types": "DRAWN, UPLOADED",
        "signature_color": "#290088",
        "sender_name": "Company Name",
        "sender_enterprise": null
      },
      "signature": {
        "style": "SIMPLE",
        "type": "INTERNATIONAL",
        "image": "DEFAULT",
        "text": "DEFAULT",
        "bg_image": "https://example.com/signature-bg.png"
      },
      "validations": {
        "email": {
          "order": 0,
          "enabled": "FALSE"
        },
        "phone": {
          "order": 1,
          "enabled": "FALSE",
          "methods": "SMS"
        },
        "id_check": {
          "order": 2,
          "enabled": "FALSE"
        },
        "biometric": {
          "order": 3,
          "enabled": "FALSE"
        }
      }
    }
  }
}
```

**Note**: The `envelope.evidence_generated` event includes:

- `evidence_document_url`: Direct link to download the evidence sheet PDF
- `evidence_generated_at`: Timestamp when the evidence was generated

These fields are available at the top level of the data object. The payload also includes complete
envelope information: documents, recipients, signature placements, and all configuration settings.

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
