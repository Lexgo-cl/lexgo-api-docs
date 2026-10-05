# Settings API

The Settings API allows you to configure default settings for your API key. Instead of sending custom settings with every envelope creation, set defaults here and override them per envelope as needed.

!!! tip "Per-API Key Configuration"
    Settings are stored at the API key level. If you need multiple configuration sets, create separate API keys for each.

## Overview

Configure default settings for:
- **Branding**: Company logo, colors, and URLs
- **Documents**: ID placement and custom text
- **Emails**: Notification preferences
- **Localization**: Multi-language content
- **Signature**: Certificate type and visual style
- **Signature Flow**: Language, order, and signature types

---

## Get Settings

Retrieve current configuration settings.

**Endpoint:** `GET /v1/settings`

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `scope` | string | Comma-separated list of setting categories to retrieve. Omit to get all settings. |

**Allowed scope values**: `documents`, `emails`, `signature_flow`, `signature`, `validations`, `branding`, `localization`

### Request

=== "All Settings"
    ```bash
    curl https://api.lexgo.cl/v1/settings \
      -H "Authorization: YOUR_API_KEY"
    ```

=== "Filtered by Scope"
    ```bash
    curl "https://api.lexgo.cl/v1/settings?scope=documents,emails,signature_flow" \
      -H "Authorization: YOUR_API_KEY"
    ```

### Response

```json
{
  "settings": {
    "branding": {
      "logo": "https://example.com/logo.png",
      "url_homepage": "https://example.com",
      "url_signature_flow": "sign.example.com",
      "name": "Your Company",
      "reply_to": "support@example.com",
      "sender_email": "noreply@example.com"
    },
    "documents": {
      "include_page_id": "TRUE",
      "custom_page_id": "Document ID:"
    },
    "emails": {
      "envelope_invitation": "TRUE",
      "envelope_voided": "FALSE",
      "envelope_rejected": "FALSE",
      "envelope_signed": "TRUE"
    },
    "localization": {
      "en": {
        "extra_page_intro": "IN WITNESS WHEREOF..."
      },
      "es": {
        "extra_page_intro": "EN TESTIMONIO DE LO CUAL...",
        "emails": {
          "email_validation": {
            "body": "<p>Custom HTML content</p>"
          }
        }
      }
    },
    "signature": {
      "type": "INTERNATIONAL",
      "style": "SIMPLE",
      "bg_image": "https://example.com/signature-bg.png"
    },
    "signature_flow": {
      "language": "EN",
      "sender_name": "Lexgo",
      "signature_order": "PARALLEL",
      "signature_types": "WRITTEN,DRAWN,UPLOADED",
      "signature_color": "#000000"
    },
    "validations": {
      "email": {
        "enabled": "FALSE",
        "order": 0
      }
    }
  },
  "request_id": "req_abc123"
}
```

---

## Update Settings

Update default configuration settings.

**Endpoint:** `PUT /v1/settings`

### Request

```bash
curl -X PUT https://api.lexgo.cl/v1/settings \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "branding": {
      "logo": "https://example.com/logo.png",
      "name": "Your Company"
    },
    "signature_flow": {
      "language": "ES",
      "signature_order": "SEQUENTIAL"
    }
  }'
```

### Response

```json
{
  "success": true,
  "settings": {
    "branding": {...},
    "signature_flow": {...}
  },
  "request_id": "req_def456"
}
```

---

## Configuration Options

### Branding Settings

Customize company branding and contact information.

| Field | Type | Description |
|-------|------|-------------|
| `branding[logo]` | string | Logo URL or base64. Use `DEFAULT` to reset. Recommended size: 100x50 pixels |
| `branding[url_homepage]` | string | URL to redirect when logo is clicked |
| `branding[url_signature_flow]` | string | Custom TLS domain for signature flows (contact support to enable) |
| `branding[name]` | string | Company name shown in emails and signature flow |
| `branding[reply_to]` | string | Reply-to email address for envelope invitations |
| `branding[sender_email]` | string | From email address for envelope invitations |

**Example:**

```json
{
  "branding": {
    "logo": "https://cdn.example.com/logo.png",
    "url_homepage": "https://www.example.com",
    "url_signature_flow": "sign.example.com",
    "name": "ACME Corporation",
    "reply_to": "support@example.com",
    "sender_email": "noreply@example.com"
  }
}
```

!!! note "Custom Domain"
    Contact support to enable a custom TLS domain for `url_signature_flow`. DNS validation and routing instructions will be provided.

---

### Document Settings

Configure document ID placement and custom text.

| Field | Type | Description |
|-------|------|-------------|
| `documents[include_page_id]` | string | Write Document ID on every page. Values: `TRUE` (default), `FALSE` |
| `documents[custom_page_id]` | string | Text that precedes the Document ID. Defaults to `"Lexgo Sign ID:"` |

**Example:**

```json
{
  "documents": {
    "include_page_id": "TRUE",
    "custom_page_id": "CVE:"
  }
}
```

**Result**: Each page will display `CVE: ABC-123-DEF-456`

---

### Email Notification Settings

Control which email notifications are sent automatically.

| Field | Type | Description |
|-------|------|-------------|
| `emails[envelope_invitation]` | string | Send invitation emails. Values: `TRUE` (default), `FALSE` |
| `emails[envelope_voided]` | string | Notify when envelope is voided. Values: `TRUE` (default), `FALSE` |
| `emails[envelope_rejected]` | string | Notify when envelope is rejected. Values: `TRUE` (default), `FALSE` |
| `emails[envelope_signed]` | string | Notify when envelope is signed. Values: `TRUE` (default), `FALSE` |

**Example:**

```json
{
  "emails": {
    "envelope_invitation": "TRUE",
    "envelope_voided": "FALSE",
    "envelope_rejected": "FALSE",
    "envelope_signed": "TRUE"
  }
}
```

!!! tip "Production Use"
    In production, disable test notifications and only enable essential emails to reduce noise.

---

### Localization Settings

Configure multi-language content for emails and documents.

#### Extra Page Text

Text displayed on extra signature pages (when multiple signers).

| Field | Type | Description |
|-------|------|-------------|
| `localization[en][extra_page_intro]` | string | English intro text for extra pages |
| `localization[es][extra_page_intro]` | string | Spanish intro text for extra pages |

**Example:**

```json
{
  "localization": {
    "en": {
      "extra_page_intro": "IN WITNESS WHEREOF, the parties hereto have subscribed this Contract to be effective as of this date."
    },
    "es": {
      "extra_page_intro": "EN TESTIMONIO DE LO CUAL, las partes suscriben este Contrato para que surta efectos a partir de esta fecha."
    }
  }
}
```

#### Custom Email Templates

Customize email validation content per language.

**Example:**

```json
{
  "localization": {
    "es": {
      "emails": {
        "email_validation": {
          "body": "<p>Hola {{recipient.name}},</p><p>Ingresa este código: <strong>{{validations.code}}</strong></p>"
        }
      }
    }
  }
}
```

**Available Variables**:
- `{{recipient.name}}` - Recipient name
- `{{validations.code}}` - 6-digit verification code
- `{{envelope.name}}` - Envelope name

See [Custom Email Templates](../guides/email-templates.md) for more details.

---

### Signature Settings

Configure digital signature certificate type and appearance.

| Field | Type | Description |
|-------|------|-------------|
| `signature[type]` | string | Signing type: `INTERNATIONAL` (default, FES) or `QUALIFIED_CHILE` (FEA) |
| `signature[style]` | string | Signature appearance: `SIMPLE` (default), `SIMPLE_QR`, `FULL` |
| `signature[bg_image]` | string | Signature-area image: `DEFAULT` (Lexgo logo), `NONE` (disabled), or a custom image URL |
| `signature[bg_image_scope]` | string | `QUALIFIED_CHILE` (default), `INTERNATIONAL`, `ALL`; legacy `FEA`/`FES` aliases |

**Example:**

```json
{
  "signature": {
    "type": "QUALIFIED_CHILE",
    "style": "FULL",
    "bg_image": "https://cdn.example.com/signature-bg.png"
  }
}
```

**Signing type and appearance are independent.** `signature.type` chooses
International (FES) or Qualified Chile (FEA) signing. `signature.style` changes
the visible signature layout; `SIMPLE`, `SIMPLE_QR`, and `FULL` are styles,
not signing types.

#### Signature Background Image

`signature.bg_image_scope` chooses which signing type draws the background.
It does not change the signing type, certificate, or legal status.

| Scope | Qualified Chile (`QUALIFIED_CHILE`, FEA) | International (`INTERNATIONAL`, FES) |
|-------|-------------------------------------------|---------------------------------------|
| `QUALIFIED_CHILE` (default) | Background enabled | Background disabled |
| `INTERNATIONAL` | Background disabled | Background enabled |
| `ALL` | Background enabled | Background enabled |

Legacy values `FEA` and `FES` remain accepted aliases for
`QUALIFIED_CHILE` and `INTERNATIONAL`, respectively. Use the canonical
values in new integrations. Unrecognized values are retained in the settings response, but their effective
rendering scope falls back to `QUALIFIED_CHILE`. Accepted legacy aliases are
normalized to canonical values in settings readback and new updates.

`bg_image: "NONE"` disables the signature background for every scope.
`DEFAULT` uses the Lexgo logo; a custom image URL selects that image.
The image is composited behind the signature with background opacity.

**Precedence:** a nonblank per-envelope value under `settings.signature`
overrides that field's API-key default. An omitted field uses the API-key
default, then the built-in default (`DEFAULT` image,
`QUALIFIED_CHILE` scope). Resolved settings are frozen when the envelope is
sent; later default changes do not change already-frozen envelopes.
Defaults can still affect unsent envelopes that omit an override.

Set a persistent default:

```bash
curl -X PUT https://api.lexgo.cl/v1/settings \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"signature":{"bg_image":"DEFAULT","bg_image_scope":"ALL"}}'
```

Override it for one envelope by including:

```json
{"settings":{"signature":{"type":"INTERNATIONAL","style":"SIMPLE","bg_image_scope":"INTERNATIONAL"}}}
```

The sandbox `NON-BINDING DOCUMENT` marking on document pages is independent
of this signature-area background. Disabling `bg_image` never removes the
sandbox marking. Qualified Chile sandbox signing follows the genuine provider
journey and team-recipient requirements in the [Sandbox guide](../guides/sandbox.md).
The background scope changes appearance; select `signature.type` separately
to choose the signing journey.

---

### Signature Flow Settings

Configure the signing experience and recipient workflow.

| Field | Type | Description |
|-------|------|-------------|
| `signature_flow[language]` | string | Default language: `EN` (default), `ES` |
| `signature_flow[sender_name]` | string | Name of sender shown in emails. Defaults to `"Lexgo"` |
| `signature_flow[signature_order]` | string | Signing order: `PARALLEL` (default), `SEQUENTIAL` |
| `signature_flow[signature_types]` | string | Allowed signature methods: `WRITTEN`, `DRAWN`, `UPLOADED` (comma-separated) |
| `signature_flow[signature_color]` | string | Hex color for drawn signatures. Must start with `#`. Defaults to `"#000000"` |

**Example:**

```json
{
  "signature_flow": {
    "language": "ES",
    "sender_name": "ACME Legal",
    "signature_order": "SEQUENTIAL",
    "signature_types": "DRAWN,UPLOADED",
    "signature_color": "#290088"
  }
}
```

**Signature Order**:
- `PARALLEL`: All recipients can sign simultaneously
- `SEQUENTIAL`: Recipients sign in order specified by `order` field

**Signature Types**:
- `WRITTEN`: Type signature with keyboard
- `DRAWN`: Draw signature with mouse/touchscreen
- `UPLOADED`: Upload signature image file

---

### Validation Settings

Configure default email validation (2FA) behavior.

| Field | Type | Description |
|-------|------|-------------|
| `validations[email][enabled]` | string | Enable email validation: `TRUE`, `FALSE` (default) |
| `validations[email][order]` | integer | When to validate: `0` (before document access), `1` (after viewing) |

**Example:**

```json
{
  "validations": {
    "email": {
      "enabled": "TRUE",
      "order": 0
    }
  }
}
```

**Validation Order**:
- `0`: Validate before allowing document access (recommended for sensitive documents)
- `1`: Validate after viewing document but before signing

---

## Complete Example

### Request

```bash
curl -X PUT https://api.lexgo.cl/v1/settings \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "branding": {
      "logo": "https://cdn.example.com/logo-100x50.png",
      "url_homepage": "https://www.example.com",
      "name": "ACME Corporation",
      "reply_to": "support@example.com",
      "sender_email": "noreply@example.com"
    },
    "documents": {
      "include_page_id": "TRUE",
      "custom_page_id": "DOC-ID:"
    },
    "emails": {
      "envelope_invitation": "TRUE",
      "envelope_voided": "TRUE",
      "envelope_rejected": "FALSE",
      "envelope_signed": "TRUE"
    },
    "signature_flow": {
      "language": "ES",
      "sender_name": "ACME Legal",
      "signature_order": "SEQUENTIAL",
      "signature_types": "DRAWN,UPLOADED",
      "signature_color": "#0066cc"
    },
    "signature": {
      "type": "QUALIFIED_CHILE",
      "style": "FULL"
    },
    "validations": {
      "email": {
        "enabled": "TRUE",
        "order": 0
      }
    },
    "localization": {
      "es": {
        "extra_page_intro": "EN TESTIMONIO DE LO CUAL, las partes suscriben este Contrato."
      }
    }
  }'
```

---

## Overriding Per Envelope

Settings configured here are **defaults**. Override them per envelope by including the same parameters in the envelope creation request.

**Example: Override sender name for specific envelope**

```json
{
  "name": "Employment Contract",
  "documents": [...],
  "recipients": [...],
  "settings": {
    "signature_flow": {
      "sender_name": "HR Department"
    }
  }
}
```

The envelope will use "HR Department" as sender name instead of the default "ACME Legal".

---

## Best Practices

### Development vs Production

Create separate API keys for different environments:

```json
// Development key settings
{
  "emails": {
    "envelope_invitation": "FALSE",
    "envelope_signed": "FALSE"
  },
  "signature_flow": {
    "language": "EN"
  }
}

// Production key settings
{
  "emails": {
    "envelope_invitation": "TRUE",
    "envelope_signed": "TRUE"
  },
  "signature_flow": {
    "language": "ES"
  }
}
```

### Branding Consistency

- Use consistent logo dimensions (100x50 pixels)
- Match signature color to your brand
- Set proper reply-to addresses for support inquiries

### Email Optimization

- Disable unnecessary notifications to reduce email volume
- Only enable essential alerts in production
- Use webhooks for programmatic event handling instead of emails

---

## Troubleshooting

### Logo Not Displaying

**Cause**: Logo URL not publicly accessible or wrong format.

**Solution**:
- Ensure URL is public (not behind authentication)
- Use HTTPS URLs
- Supported formats: PNG, JPG, SVG
- Recommended size: 100x50 pixels

### Custom Domain Not Working

**Cause**: DNS not configured or TLS not enabled.

**Solution**:
- Contact support to enable custom domain feature
- Follow provided DNS validation instructions
- Allow 24-48 hours for DNS propagation

### Settings Not Applied

**Cause**: Envelope-level settings overriding defaults.

**Solution**:
- Check if settings are passed in envelope creation request
- Envelope-level settings always take precedence
- Remove envelope-level settings to use defaults

---

## Related Documentation

- [Custom Email Templates](../guides/email-templates.md) - Customize email content
- [Email Validation (2FA)](../guides/email-validation.md) - Configure 2FA
- [Envelopes API](envelopes.md) - Override settings per envelope

---

## Rate Limits

Settings API has no rate limits. However, frequent updates are not recommended as they affect all new envelopes created with that API key.
