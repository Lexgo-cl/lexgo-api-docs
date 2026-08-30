# Signing Reminders (Client API)

Schedule an email reminder for a signing document. This endpoint belongs to the
**Client API** (`/api/client/…`), not the LexgoSign `/api/v1` envelope surface.

!!! info "API surface"
    Client API uses **OAuth** (`Bearer` token, `client` scope) and requires the
    `X-Enterprise-Id` header. See authentication notes below — this differs from
    LexgoSign API keys used by `/api/v1/envelopes`.

Internally, the call creates an `Alerts::Alert` (type `custom`) and an
`Alerts::Reminder`. Delivery is handled by the scheduled `ReminderWorker`.

## Endpoint

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/api/client/documents/:id/reminders` | Schedule an email reminder for a document |

**Required permission:** `contracts:write`

## Authentication

| Header | Value |
|--------|-------|
| `Authorization` | `Bearer <oauth_access_token>` (Doorkeeper, `client` scope) |
| `X-Enterprise-Id` | UUID of the target enterprise |
| `Content-Type` | `application/json` or `application/x-www-form-urlencoded` |

!!! note "Gateway path"
    On `api.lexgo.cl` / `api-beta.lexgo.cl`, API Gateway strips the `/api` prefix.
    Call `https://api.lexgo.cl/client/documents/:id/reminders` externally; Rails
    still mounts the route as `/api/client/…`.

## Path parameters

| Param | Type | Description |
|-------|------|-------------|
| `id` | UUID | Document (`DocumentResolved`) ID, scoped to the enterprise |

## Request body

| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `alert_date` | ISO date `YYYY-MM-DD` | yes | Anchor date (e.g. signing deadline) |
| `offset_days` | positive integer ≥ 1 | yes | Days before `alert_date` to send the reminder |
| `channel` | string | no | Only `"email"` is supported (default: `"email"`) |
| `custom_message` | string | no | Optional free-text included in the reminder email |

### Computed `fire_date`

```
fire_date = alert_date - offset_days
```

`fire_date` must be **today or in the future** at creation time. If it would
already be in the past, the API returns `422`.

## Example request

=== "cURL"
    ```bash
    curl -X POST "https://api.lexgo.cl/client/documents/DOCUMENT_UUID/reminders" \
      -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
      -H "X-Enterprise-Id: YOUR_ENTERPRISE_UUID" \
      -H "Content-Type: application/json" \
      -d '{
        "alert_date": "2026-09-01",
        "offset_days": 3,
        "channel": "email",
        "custom_message": "Please sign before the deadline"
      }'
    ```

=== "Python"
    ```python
    import requests

    response = requests.post(
        "https://api.lexgo.cl/client/documents/DOCUMENT_UUID/reminders",
        headers={
            "Authorization": "Bearer YOUR_ACCESS_TOKEN",
            "X-Enterprise-Id": "YOUR_ENTERPRISE_UUID",
            "Content-Type": "application/json",
        },
        json={
            "alert_date": "2026-09-01",
            "offset_days": 3,
            "custom_message": "Please sign before the deadline",
        },
    )
    print(response.status_code, response.json())
    ```

=== "Ruby"
    ```ruby
    require "net/http"
    require "json"
    require "uri"

    uri = URI("https://api.lexgo.cl/client/documents/DOCUMENT_UUID/reminders")
    http = Net::HTTP.new(uri.host, uri.port)
    http.use_ssl = true

    request = Net::HTTP::Post.new(uri)
    request["Authorization"] = "Bearer YOUR_ACCESS_TOKEN"
    request["X-Enterprise-Id"] = "YOUR_ENTERPRISE_UUID"
    request["Content-Type"] = "application/json"
    request.body = {
      alert_date: "2026-09-01",
      offset_days: 3,
      custom_message: "Please sign before the deadline",
    }.to_json

    response = http.request(request)
    puts response.code, response.body
    ```

## Success response — `201 Created`

```json
{
  "reminder": {
    "id": 1,
    "alert_id": 42,
    "document_id": "DOCUMENT_UUID",
    "alert_date": "2026-09-01",
    "offset_days": 3,
    "fire_date": "2026-08-29",
    "channel": "email",
    "delivery_status": "pending",
    "created_at": "2026-08-05T17:00:00Z"
  }
}
```

| Field | Description |
|-------|-------------|
| `id` | Reminder ID |
| `alert_id` | Parent alert ID |
| `document_id` | Document UUID |
| `alert_date` | Anchor date you supplied |
| `offset_days` | Days before `alert_date` |
| `fire_date` | Date the email will be sent (`alert_date - offset_days`) |
| `channel` | Delivery channel (`email`) |
| `delivery_status` | `pending` at creation; see Delivery below |
| `created_at` | ISO-8601 timestamp |

## Delivery

1. API creates an alert (`custom`) and a pending reminder; the OAuth resource owner is subscribed.
2. On `fire_date`, `ReminderWorker` selects pending reminders due that day.
3. Subscribed users with enterprise access receive the email; status becomes `sent` or `failed`.

`delivery_status` values: `pending`, `sent`, `failed`, `expired`. The worker sets
`sent` / `failed`. `expired` is set internally when an alert's `alert_date` moves
into the past (e.g. edited in the Lexgo UI after the reminder was scheduled) — it
is not produced by this create endpoint.

!!! tip "Document state"
    Reminder creation is allowed regardless of signing status (pending, signed, voided).
    Callers should confirm the document is still unsigned before scheduling.

## Errors

| Status | Condition |
|--------|-----------|
| `400` | `alert_date` missing or not a valid `YYYY-MM-DD` date |
| `400` | `offset_days` missing, zero, or negative |
| `400` | `channel` is not `"email"` |
| `400` | `X-Enterprise-Id` header missing |
| `401` | Missing or expired OAuth token |
| `403` | Enterprise not authorized on token, or missing `contracts:write` |
| `404` | Document not found in the enterprise scope |
| `422` | Computed `fire_date` is in the past |
| `422` | Model validation failure (`details` array in body) |

### Example error body

```json
{
  "error": "fire_date (alert_date minus offset_days) is in the past"
}
```

## Out of scope (current version)

- Listing, updating, or deleting reminders via API
- Non-email channels (WhatsApp, in-app notification)

## Related

- [API Overview](overview.md)
- [Example: Schedule a signing reminder](../examples/schedule-signing-reminder.md)
- [Envelopes API](envelopes.md) (LexgoSign `/api/v1` surface)
