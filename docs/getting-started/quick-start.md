# Quick Start

## Prepare Your Integration

Create an [API key](authentication.md), select sandbox or live stage, and
prepare a PDF and recipient. Both stages use `https://api.lexgo.cl/v1`.
For sandbox, recipients must be enterprise team members and Qualified Chile
signing remains restricted; see [Sandbox vs Live](../guides/sandbox.md).

## Create an Envelope

`POST /v1/envelopes` accepts JSON or multipart/form-data with nested bracket
notation. Provide `documents`, `recipients`, and `placements` keyed by your
identifiers. Every recipient and original document needs a placement.
A SIGNATURE placement can omit coordinates/token for an added signature page.
For a positioned signature, use `coordinates.top`, `coordinates.left`, and
zero-based `coordinates.page`, or a PDF text `token`.

See the [complete create/send example](../examples/create-envelope.md).
Inspect the response's `envelope.id`, `status`, `errors`, and `warnings`.
Create returns HTTP 200 for a valid envelope, or HTTP 422 with
`status: ERROR` and actionable `envelope.errors`. Preserve the returned ID
and fix the envelope before sending.
Keep its ID for reconciliation, including after a network timeout.

## Send the Envelope

Only a valid `CREATED` envelope can be sent.

```bash
curl -X PUT https://api.lexgo.cl/v1/envelopes/ENVELOPE_ID/send \
  -H "Authorization: YOUR_API_KEY"
```

The response is HTTP **202 Accepted** with `success: true` and an envelope
in `SENDING`. Processing runs asynchronously. This does not prove email
delivery or signature completion. When invitation email is disabled, retrieve
recipient signing links from the envelope response.

## Track Completion and Errors

Use [webhooks](../guides/webhook-integration.md) for prompt updates and
`GET /v1/envelopes/ENVELOPE_ID` for polling/reconciliation. Delivery is best
effort, so missed notifications must not leave your integration waiting forever.

| Status | Meaning |
|--------|---------|
| `CREATED` | Not sent; fix validation errors before send |
| `SENDING` | Asynchronous processing is underway |
| `IN_PROGRESS` | Signing workflow is open |
| `SUCCESS` | Envelope completed |
| `VOIDED` | Envelope canceled |
| `ERROR` | Inspect `errors` and investigate or correct the envelope |

Handle `envelope.sending_failed` notifications and reconcile state rather
than blindly repeating create/send. Treat a client timeout as an unknown
result until you inspect the existing envelope.

## Download Evidence

After `SUCCESS`, call:

```bash
curl https://api.lexgo.cl/v1/envelopes/ENVELOPE_ID/evidence \
  -H "Authorization: YOUR_API_KEY"
```

Inspect `success` and `evidence_document_url`; an unsuccessful generation
can return HTTP 500. Other envelope states return 405.

- [Envelopes API](../api/envelopes.md)
- [Settings and signature backgrounds](../api/settings.md#signature-background-image)
- [Email validation (2FA)](../guides/email-validation.md)
- [Webhook Integration](../guides/webhook-integration.md)
