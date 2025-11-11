# Example: Email Validation Flow

Complete example implementing email validation (2FA).

## Code Example

```python
import requests

API_KEY = 'your_api_key_here'
BASE_URL = 'https://api.lexgo.cl/api/v1'
ENVELOPE_ID = 'your_envelope_id'
RECIPIENT_ID = 'your_recipient_id'

# Send verification code
response = requests.post(
    f'{BASE_URL}/envelopes/{ENVELOPE_ID}/recipients/{RECIPIENT_ID}/validations/send_code',
    headers={'Authorization': API_KEY}
)

if response.status_code == 200:
    print(f"Code sent! Expires: {response.json()['expires_at']}")
else:
    print(f"Error: {response.json()['error']}")
    exit(1)

# Get code from user
code = input("Enter 6-digit code: ")

# Verify code
response = requests.post(
    f'{BASE_URL}/envelopes/{ENVELOPE_ID}/recipients/{RECIPIENT_ID}/validations/verify_code',
    headers={'Authorization': API_KEY},
    json={'code': code}
)

if response.status_code == 200:
    print("✓ Email validated!")
else:
    print(f"✗ Failed: {response.json()['error']}")
```

## Next Steps

See [Email Validation Guide](../guides/email-validation.md) for full implementation details.
