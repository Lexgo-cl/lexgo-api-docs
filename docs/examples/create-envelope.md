# Example: Create and Send Envelope

Complete example showing how to create and send an envelope.

## Code Example

```python
import requests
import base64

API_KEY = 'your_api_key_here'
BASE_URL = 'https://api.lexgo.cl/api/v1'

# Read and encode PDF
with open('contract.pdf', 'rb') as f:
    pdf_base64 = base64.b64encode(f.read()).decode('utf-8')

# Create envelope using form-data
form_data = {
    'name': 'Employment Contract - John Doe',
    'documents[0][base64]': pdf_base64,
    'documents[0][name]': 'contract.pdf',
    'recipients[0][name]': 'John Doe',
    'recipients[0][email]': 'john.doe@example.com',
    'recipients[0][order]': '1'
}

response = requests.post(
    f'{BASE_URL}/envelopes',
    headers={'Authorization': API_KEY},
    data=form_data
)

envelope_id = response.json()['envelope']['id']
print(f"Created: {envelope_id}")

# Send envelope
response = requests.post(
    f'{BASE_URL}/envelopes/{envelope_id}/send_invitation',
    headers={'Authorization': API_KEY}
)

print(f"Sent: {response.json()['success']}")
```

## Next Steps

See [Quick Start Guide](../getting-started/quick-start.md) for full walkthrough.
