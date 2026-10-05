# Webhook Integration Guide

Subscribe through the [Webhooks API](../api/webhooks.md). Subscriptions are
comma-separated exact event names, for example
`envelope.success,envelope.sending_failed,recipient.signed_all`.

## Receive and Acknowledge

1. Use an HTTPS endpoint and configure `auth_type: "header"` plus an
   `auth` secret. Verify the exact Authorization header before processing.
2. Persist the delivery `id` and payload durably before returning a 2xx response.
3. Process the stored payload asynchronously. Deduplicate by delivery `id`
   and make business operations idempotent by resource ID.
4. Acknowledge already-stored duplicate IDs with 2xx.

Retries reuse the delivery `id`; independent events or webhook subscriptions
can have different IDs for the same resource. Do not assume one webhook
corresponds to one signing action or that events arrive in order.

## Best-Effort Delivery

Failures and timeouts have up to three scheduled retries after approximately
5, 10, and 20 seconds. A receiver timeout can occur after you already accepted
a payload, so duplicates are expected. Delivery is neither guaranteed nor
exactly once.

The `data` object reflects resource state when the attempt is sent. It can
change across retries. Use `event_type` to identify the notification and
retrieve current envelope state before irreversible business actions.

## Reconcile with the API

Keep envelope IDs from creation. Use `GET /v1/envelopes/:id` periodically and
after a missed event or interrupted send request to reconcile your local state.
Inspect `status`, `errors`, and `warnings`; HTTP 200 is not a signing success.
Track `SENDING` until processing succeeds or fails. Route `ERROR` and
`envelope.sending_failed` for investigation rather than waiting indefinitely.

Use notifications for prompt updates and polling for recovery. This guidance
does not claim that every upstream callback reaches your endpoint.

- [Webhooks API Reference](../api/webhooks.md)
- [Envelope lifecycle](../api/envelopes.md)
- [Example: Webhook Subscriptions](../examples/webhook-subscriptions.md)
