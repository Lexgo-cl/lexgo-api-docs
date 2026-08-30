# Example: Webhook Subscriptions

Complete example for subscribing to webhooks.

## Code Example

```python
import requests

API_KEY = 'your_api_key_here'
BASE_URL = 'https://api.lexgo.cl/api/v1'

# Create webhook
response = requests.post(
    f'{BASE_URL}/webhooks',
    headers={
        'Authorization': API_KEY,
        'Content-Type': 'application/json'
    },
    json={
        'url': 'https://your-app.com/webhooks/lexgo',
        'subscriptions': 'envelope.success,recipient.signed_all,recipient.validation_code_success',
        'auth_type': 'header',
        'auth': 'Bearer your_webhook_secret'
    }
)

webhook = response.json()['webhook']
print(f"Created webhook: {webhook['id']}")
```

## Webhook Handler

```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/webhooks/lexgo', methods=['POST'])
def handle_webhook():
    payload = request.json
    event_type = payload['event_type']
    data = payload['data']

    if event_type == 'envelope.success':
        print(f"Envelope {data['id']} completed!")

    return {'success': True}, 200
```

## Next Steps

See [Webhooks API](../api/webhooks.md) for full reference.
