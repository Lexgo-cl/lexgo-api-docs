# Example: Schedule a Signing Reminder

End-to-end Client API flow: obtain a document ID, schedule a reminder, and
understand when the email is sent.

## Prerequisites

- OAuth access token with `client` scope
- Enterprise UUID your token is authorized for
- A document UUID in that enterprise (unsigned signing document recommended)

## 1. Schedule the reminder

```bash
DOCUMENT_ID="your-document-uuid"
TOKEN="your_oauth_access_token"
ENTERPRISE_ID="your-enterprise-uuid"
ALERT_DATE=$(date -u -d "+30 days" +%Y-%m-%d)   # macOS: date -u -v+30d +%Y-%m-%d

curl -i -X POST "https://api.lexgo.cl/client/documents/${DOCUMENT_ID}/reminders" \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "X-Enterprise-Id: ${ENTERPRISE_ID}" \
  -H "Content-Type: application/json" \
  -d "{
    \"alert_date\": \"${ALERT_DATE}\",
    \"offset_days\": 3,
    \"custom_message\": \"Please sign before the deadline\"
  }"
```

Expected: **HTTP 201** with a `reminder` object. Check `fire_date` equals
`alert_date` minus `offset_days`.

## 2. What was created

| Record | Meaning |
|--------|---------|
| `Alerts::Alert` (`custom`) | Parent alert anchored to `alert_date` |
| `Alerts::Reminder` (`pending`) | Scheduled delivery with `offset_days` |
| Alert subscription | OAuth resource owner receives the email |

## 3. Delivery on `fire_date`

On `fire_date`, `ReminderWorker` sends the email to subscribed users who still
have access to the enterprise, then marks the reminder `sent` or `failed`.

To verify on staging:

1. Create a reminder with `alert_date = today + offset_days` so `fire_date` is **today**.
2. Trigger or wait for `ReminderWorker#send_alert_reminders`.
3. Confirm outbound mail (on staging, the interceptor routes all mail to
   `contacto@beta.lexgo.cl` — see staging testing guide).

## 4. Common failures to try

| Request | Expected |
|---------|----------|
| Omit `alert_date` | `400` |
| `offset_days: 0` | `400` |
| `channel: "whatsapp"` | `400` |
| `alert_date` in the past such that `fire_date < today` | `422` |
| Document from another enterprise | `404` |
| Token without `contracts:write` | `403` |

## Full Python snippet

```python
from datetime import date, timedelta
import requests

BASE = "https://api.lexgo.cl"
headers = {
    "Authorization": f"Bearer {TOKEN}",
    "X-Enterprise-Id": ENTERPRISE_ID,
    "Content-Type": "application/json",
}

alert_date = (date.today() + timedelta(days=30)).isoformat()
resp = requests.post(
    f"{BASE}/client/documents/{DOCUMENT_ID}/reminders",
    headers=headers,
    json={
        "alert_date": alert_date,
        "offset_days": 3,
        "custom_message": "Please sign before the deadline",
    },
)
resp.raise_for_status()
reminder = resp.json()["reminder"]
assert reminder["fire_date"] == (date.fromisoformat(alert_date) - timedelta(days=3)).isoformat()
print("Scheduled:", reminder["id"], "fires on", reminder["fire_date"])
```

## Related

- [Signing Reminders API](../api/reminders.md)
- [API Overview](../api/overview.md)
