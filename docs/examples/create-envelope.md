# Example: Create and Send Envelope

This example uses International signing and a required placement. Omitting
coordinates/token from a SIGNATURE placement uses an added signature page.

```python
import base64
import requests

API_KEY = "your_api_key_here"
BASE_URL = "https://api.lexgo.cl/v1"
headers = {"Authorization": API_KEY}
with open("contract.pdf", "rb") as source:
    pdf_base64 = base64.b64encode(source.read()).decode("utf-8")

response = requests.post(
    f"{BASE_URL}/envelopes",
    headers=headers,
    json={
        "name": "Contract",
        "documents": {"contract": {"name": "contract.pdf", "base64": pdf_base64}},
        "recipients": {"signer": {"name": "John Doe", "email": "john@example.com"}},
        "placements": {
            "signature": {
                "document_key": "contract",
                "recipient_key": "signer",
                "type": "SIGNATURE"
            }
        },
        "settings": {"signature": {"type": "INTERNATIONAL"}}
    }
)
response.raise_for_status()
envelope = response.json()["envelope"]
envelope_id = envelope["id"]
if envelope["status"] != "CREATED" or envelope.get("errors"):
    raise RuntimeError(envelope.get("errors", envelope["status"]))

response = requests.put(f"{BASE_URL}/envelopes/{envelope_id}/send", headers=headers)
response.raise_for_status()
result = response.json()
print(response.status_code, result["envelope"]["status"])  # 202, SENDING

# Reconcile asynchronously; this response is not proof of completed signing.
response = requests.get(f"{BASE_URL}/envelopes/{envelope_id}", headers=headers)
response.raise_for_status()
envelope = response.json()["envelope"]
print(envelope["status"], envelope.get("errors", []))
```

Each recipient and original document must have a placement. An ATTACHMENT
cannot have SIGNATURE placements; use APPROVER or WITNESS when assigning one
to a recipient. See [Placements](../api/envelopes.md#placement-parameters-required).

For sandbox keys, use a recipient on your enterprise team and observe the
[Sandbox restrictions](../guides/sandbox.md).

See [Quick Start](../getting-started/quick-start.md) and
[Webhook Integration](../guides/webhook-integration.md) for completion handling.
