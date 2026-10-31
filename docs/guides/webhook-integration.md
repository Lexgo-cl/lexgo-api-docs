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

## Source IP Allowlisting

As an additional control, your receiver firewall or WAF can restrict incoming
HTTPS `POST` requests on the webhook endpoint to the Lexgo webhook source IPv4
addresses below. These are the addresses Lexgo sends webhooks *from*; they are
not used for, and do not restrict, requests you send *to* the Lexgo API.

| Environment | Source IPv4 (allow each as `/32`) |
|-------------|-----------------------------------|
| Production, address 1 | `3.148.9.93/32` |
| Production, address 2 | `18.221.200.15/32` |
| Staging (`api-beta.lexgo.cl`) | `54.243.84.24/32` |

- Allow **both** production addresses: any production delivery, including
  retries, can come from either one.
- Webhooks for sandbox and live API keys on `api.lexgo.cl` are production
  deliveries. Staging is a separate Lexgo deployment, not the sandbox stage;
  see [Sandbox vs staging](sandbox.md#sandbox-vs-staging).
- Allow only the individual addresses for your environment, not cloud
  provider ranges or the addresses of Lexgo API load balancers.

An allowlist does not authenticate a webhook or its payload. Keep the HTTPS
endpoint and verify the `auth_type: "header"` Authorization value as described
above.

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
