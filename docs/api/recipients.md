# Recipients API

Recipients are the individuals who sign, approve, or receive documents in an envelope.

!!! info "Coming Soon"
    Detailed recipient management documentation will be added in a future update.

## Overview

Recipients are defined when creating an envelope. Each recipient has:

- Unique ID
- Name and email address
- Signing order
- Role (signer, approver, recipient)
- Status tracking
- Access token for security

## Creating Recipients

Recipients are created as part of the envelope creation process:

```json
{
  "envelope": {
    "envelope_signers": [
      {
        "name": "John Doe",
        "email": "john.doe@example.com",
        "order": 1
      }
    ]
  }
}
```

See [Envelopes API](envelopes.md) for full details.

## Recipient Status

| Status | Description |
|--------|-------------|
| `PENDING` | Awaiting their turn |
| `EMAIL_SENT` | Invitation email sent |
| `IN_PROGRESS` | Currently able to sign |
| `SIGNED` | Completed signing |
| `REJECTED` | Declined to sign |

## Next Steps

- [Envelopes API](envelopes.md) - Create envelopes with recipients
- [Validations API](validations.md) - Email validation for recipients
