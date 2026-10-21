# Recipients API

Recipients are the individuals who sign, approve, or receive documents in an envelope.

!!! info "Coming Soon"
    Detailed recipient management documentation will be added in a future update.

## Overview

Recipients are defined when creating an envelope (and replaced as a whole when updating it). Each recipient has:

- Unique ID and your own `custom_key` (the `key` you used in the request)
- Name and email address
- Signing order
- Status tracking
- A private signing link, returned as `private_signature_link` once the status is `LINK_GENERATED`

Roles (signature, approver, witness) are assigned through `placements[key][type]`, not on the recipient.

## Creating Recipients

Recipients are created as part of the envelope creation process:

Send `recipients[key][field]` at the top level of the request, where `key` is any string you choose (a numeric index or a custom identifier that placements reference):

```http
recipients[0][name]=John Doe
recipients[0][email]=john.doe@example.com
recipients[0][order]=1
```

In JSON, the same data is a top-level `recipients` object keyed by `key`:

```json
{
  "recipients": {
    "0": { "name": "John Doe", "email": "john.doe@example.com", "order": 1 }
  }
}
```

See [Envelopes API](envelopes.md) for full details.

## Recipient Status

Each recipient in an envelope response has a `status` (uppercase). Every status change also fires a `recipient.<status>` [webhook event](webhooks.md#recipient-events).

| Status | Description |
|--------|-------------|
| `CREATED` | Recipient added, envelope not yet sent |
| `LINK_GENERATED` | Signing link generated (returned when the envelope's invitation email is disabled) |
| `EMAIL_SENT` | Invitation email sent |
| `EMAIL_RECEIVED` | Invitation email delivered |
| `EMAIL_OPENED` | Recipient opened the email |
| `EMAIL_CLICKED` | Recipient clicked the link in the email |
| `EMAIL_BOUNCED` | Invitation email bounced |
| `IN_PROGRESS` | Currently able to sign |
| `SIGNED_ALL` | Completed signing |
| `REJECTED` | Declined to sign |

## Next Steps

- [Envelopes API](envelopes.md) - Create envelopes with recipients
- [Validations API](validations.md) - Email validation for recipients
