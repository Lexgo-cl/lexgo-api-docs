# Email Validation (2FA) Guide

This guide walks you through implementing email-based two-factor authentication for envelope recipients.

## Overview

Email validation adds a security layer requiring recipients to verify email ownership with a 6-digit code before accessing documents.

## When to Use

Enable email validation for:

- **Sensitive contracts**: Employment agreements, NDAs
- **Legal documents**: Powers of attorney, court filings
- **High-value transactions**: Real estate contracts, M&A documents
- **Regulatory compliance**: Documents requiring identity verification

## Implementation

### Step 1: Enable in Envelope Settings

When creating an envelope, include validation settings:

```http
POST /api/v1/envelopes HTTP/1.1
Content-Type: multipart/form-data

name=Confidential Agreement
settings[validations][email][enabled]=TRUE
settings[validations][email][order]=0
recipients[0][name]=John Doe
recipients[0][email]=john.doe@example.com
recipients[0][order]=1
documents[0][base64]=JVBERi0xLjQK...
documents[0][name]=agreement.pdf
```

### Step 2: Recipient Flow

When a recipient clicks the signing link:

1. They see the validation page (not the document)
2. Click "Send Verification Code"
3. Receive email with 6-digit code
4. Enter code on validation page
5. Access document after successful verification

### Step 3: API Integration (Optional)

For custom integrations, use the Validation API:

```python
# Send code
response = requests.post(
    f'{BASE_URL}/envelopes/{envelope_id}/recipients/{recipient_id}/validations/send_code',
    headers={'Authorization': API_KEY}
)

# Verify code
response = requests.post(
    f'{BASE_URL}/envelopes/{envelope_id}/recipients/{recipient_id}/validations/verify_code',
    headers={'Authorization': API_KEY},
    json={'code': '123456'}
)
```

## Security Features

- **SHA256 Hashing**: Codes never stored in plain text
- **30-Minute Expiration**: Limited window for code use
- **Rate Limiting**: 5 codes/hour, 60-second cooldown
- **Attempt Limiting**: 3 verification attempts per code
- **Evidence Tracking**: Complete audit trail

## Customization

### Email Templates

Customize the validation email:

```json
{
  "settings": {
    "localization": {
      "en": {
        "emails": {
          "email_validation": {
            "subject": "Verify your email for {{envelope.name}}",
            "body": "Your code: {{validations.code}}",
            "cta": "Verify Email"
          }
        }
      }
    }
  }
}
```

### Template Variables

| Variable | Description |
|----------|-------------|
| `{{recipient.name}}` | Recipient name |
| `{{recipient.email}}` | Recipient email |
| `{{validations.code}}` | 6-digit code |
| `{{envelope.name}}` | Envelope name |
| `{{validations.expires_at}}` | Expiration time |

## Webhook Integration

Subscribe to validation events:

```json
{
  "subscriptions": "recipient.validation_code_sent,recipient.validation_code_success,recipient.validation_code_failed"
}
```

Track validation lifecycle in real-time.

## Best Practices

1. **Clear communication**: Inform users about email validation upfront
2. **Custom templates**: Brand validation emails to match your company
3. **Monitor failures**: Track failed validations for fraud detection
4. **Optimize UX**: Show countdown timer for code expiration
5. **Test thoroughly**: Verify email delivery in all environments

## Troubleshooting

See [Common Issues](../api/validations.md#common-issues) in the Validations API reference.

## Next Steps

- [Validations API Reference](../api/validations.md)
- [Custom Email Templates](email-templates.md)
- [Webhook Integration](webhook-integration.md)
