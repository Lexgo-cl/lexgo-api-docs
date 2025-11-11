# Custom Email Templates

Customize email communications with your branding and messaging.

!!! info "Coming Soon"
    Detailed email template customization documentation will be added in a future update.

## Overview

LexgoSign allows you to customize:

- Email subject lines
- Email body content
- Call-to-action buttons
- Multi-language support

## Template Variables

Use these variables in your templates:

| Variable | Description |
|----------|-------------|
| `{{recipient.name}}` | Recipient name |
| `{{recipient.email}}` | Recipient email |
| `{{envelope.name}}` | Document name |
| `{{validations.code}}` | Verification code (validation emails only) |
| `{{validations.expires_at}}` | Code expiration |

## Example

```json
{
  "settings": {
    "localization": {
      "en": {
        "emails": {
          "email_validation": {
            "subject": "Verify your email for {{envelope.name}}",
            "body": "Hello {{recipient.name}}, your code: {{validations.code}}"
          }
        }
      }
    }
  }
}
```

## Security

All template content is sanitized to prevent XSS attacks.
