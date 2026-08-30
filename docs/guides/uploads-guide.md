# File Uploads Guide

Step-by-step guide to upload PDF files to envelopes.

## When to Use This

Use file uploads when your PDF files are larger than 1MB. For smaller files, use the standard base64 envelope creation method.

## Quick Start

### Step 1: Create Envelope (No Documents)

```bash
curl -X POST https://api.lexgo.cl/api/v1/envelopes \
  -H "Authorization: YOUR_API_KEY" \
  -F "name=Contract - John Doe" \
  -F "recipients[0][name]=John Doe" \
  -F "recipients[0][email]=john@example.com" \
  -F "recipients[0][order]=1"
```

Save the `envelope.id` from the response.

### Step 2: Request Upload URL

```bash
curl -X POST https://api.lexgo.cl/api/v1/envelopes/ENVELOPE_ID/uploads \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file_name": "contract.pdf",
    "order": 0
  }'
```

Save the `upload.upload_url` and `upload.id`.

### Step 3: Upload File

```bash
curl -X PUT "UPLOAD_URL" \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/pdf" \
  --data-binary @contract.pdf
```

### Step 4: Wait for Processing

**Option A: Poll for completion**

```bash
curl https://api.lexgo.cl/api/v1/envelopes/ENVELOPE_ID/uploads/UPLOAD_ID \
  -H "Authorization: YOUR_API_KEY"
```

Check `upload.status` in response. Repeat every 2 seconds until `COMPLETED`.

**Option B: Use webhook (recommended for production)**

Subscribe to `envelope.file_uploaded` event:

```bash
curl -X POST https://api.lexgo.cl/api/v1/webhooks \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-app.com/webhooks/lexgo",
    "subscriptions": "envelope.file_uploaded"
  }'
```

Your webhook will receive notification when file is ready (no polling needed).

### Step 5: Verify Documents Attached

```bash
curl https://api.lexgo.cl/api/v1/envelopes/ENVELOPE_ID \
  -H "Authorization: YOUR_API_KEY"
```

Check the response:
- `envelope.documents` array contains your file
- `envelope.errors` is empty
- Each document has `status: "READY"`

### Step 6: Send Envelope

```bash
curl -X POST https://api.lexgo.cl/api/v1/envelopes/ENVELOPE_ID/send_invitation \
  -H "Authorization: YOUR_API_KEY"
```

---

## Complete Working Script

Save this as `upload_contract.sh`:

```bash
#!/bin/bash
set -e

# Configuration
API_BASE="https://api.lexgo.cl/api/v1"
API_KEY="YOUR_API_KEY"
PDF_FILE="$1"

if [ -z "$PDF_FILE" ]; then
  echo "Usage: $0 <pdf_file>"
  exit 1
fi

if [ ! -f "$PDF_FILE" ]; then
  echo "Error: File not found: $PDF_FILE"
  exit 1
fi

echo "=== Step 1: Create Envelope ==="
ENVELOPE_JSON=$(curl -s -X POST "${API_BASE}/envelopes" \
  -H "Authorization: ${API_KEY}" \
  -F "name=Contract - Test" \
  -F "recipients[0][name]=John Doe" \
  -F "recipients[0][email]=john@example.com" \
  -F "recipients[0][order]=1")

ENVELOPE_ID=$(echo "$ENVELOPE_JSON" | jq -r '.envelope.id')
if [ "$ENVELOPE_ID" = "null" ]; then
  echo "Error creating envelope:"
  echo "$ENVELOPE_JSON" | jq '.'
  exit 1
fi
echo "✓ Created envelope: $ENVELOPE_ID"

echo ""
echo "=== Step 2: Request Upload URL ==="
UPLOAD_JSON=$(curl -s -X POST "${API_BASE}/envelopes/${ENVELOPE_ID}/uploads" \
  -H "Authorization: ${API_KEY}" \
  -H "Content-Type: application/json" \
  -d "{
    \"file_name\": \"$(basename "$PDF_FILE")\",
    \"order\": 0
  }")

UPLOAD_URL=$(echo "$UPLOAD_JSON" | jq -r '.upload.upload_url')
UPLOAD_ID=$(echo "$UPLOAD_JSON" | jq -r '.upload.id')
if [ "$UPLOAD_URL" = "null" ]; then
  echo "Error requesting upload URL:"
  echo "$UPLOAD_JSON" | jq '.'
  exit 1
fi
echo "✓ Got upload URL"
echo "✓ Upload ID: $UPLOAD_ID"

echo ""
echo "=== Step 3: Upload File ==="
UPLOAD_RESPONSE=$(curl -s -X PUT "$UPLOAD_URL" \
  -H "Authorization: ${API_KEY}" \
  -H "Content-Type: application/pdf" \
  --data-binary @"$PDF_FILE")

SUCCESS=$(echo "$UPLOAD_RESPONSE" | jq -r '.success')
if [ "$SUCCESS" != "true" ]; then
  echo "Error uploading file:"
  echo "$UPLOAD_RESPONSE" | jq '.'
  exit 1
fi
echo "✓ Uploaded file"

echo ""
echo "=== Step 4: Wait for Processing ==="
MAX_ATTEMPTS=30
ATTEMPT=0
while [ $ATTEMPT -lt $MAX_ATTEMPTS ]; do
  STATUS_JSON=$(curl -s "${API_BASE}/envelopes/${ENVELOPE_ID}/uploads/${UPLOAD_ID}" \
    -H "Authorization: ${API_KEY}")

  STATUS=$(echo "$STATUS_JSON" | jq -r '.upload.status')
  echo "  Status: $STATUS (attempt $((ATTEMPT+1))/$MAX_ATTEMPTS)"

  if [ "$STATUS" = "COMPLETED" ]; then
    FILE_ID=$(echo "$STATUS_JSON" | jq -r '.upload.file_id')
    echo "✓ File processed successfully!"
    echo "✓ File ID: $FILE_ID"
    break
  elif [ "$STATUS" = "FAILED" ]; then
    ERROR=$(echo "$STATUS_JSON" | jq -r '.upload.error_message')
    echo "✗ Processing failed: $ERROR"
    exit 1
  fi

  ATTEMPT=$((ATTEMPT+1))
  sleep 2
done

if [ $ATTEMPT -eq $MAX_ATTEMPTS ]; then
  echo "✗ Timeout waiting for processing"
  exit 1
fi

echo ""
echo "=== Step 5: Verify Envelope ==="
ENVELOPE_JSON=$(curl -s "${API_BASE}/envelopes/${ENVELOPE_ID}" \
  -H "Authorization: ${API_KEY}")

DOC_COUNT=$(echo "$ENVELOPE_JSON" | jq '.envelope.documents | length')
ERRORS=$(echo "$ENVELOPE_JSON" | jq '.envelope.errors | length')

if [ "$DOC_COUNT" -eq 0 ]; then
  echo "✗ No documents attached to envelope"
  exit 1
fi

if [ "$ERRORS" -gt 0 ]; then
  echo "✗ Envelope has errors:"
  echo "$ENVELOPE_JSON" | jq '.envelope.errors'
  exit 1
fi

echo "✓ Envelope verified: $DOC_COUNT document(s), no errors"

echo ""
echo "=== Step 6: Send Envelope ==="
SEND_JSON=$(curl -s -X POST "${API_BASE}/envelopes/${ENVELOPE_ID}/send_invitation" \
  -H "Authorization: ${API_KEY}")

SUCCESS=$(echo "$SEND_JSON" | jq -r '.success')
if [ "$SUCCESS" = "true" ]; then
  echo "✓ Envelope sent!"
  echo ""
  echo "Done! Envelope ID: $ENVELOPE_ID"
else
  echo "✗ Failed to send envelope:"
  echo "$SEND_JSON" | jq '.'
  exit 1
fi
```

Make it executable and run:

```bash
chmod +x upload_contract.sh
./upload_contract.sh contract.pdf
```

---

## Python Implementation

Save as `upload_contract.py`:

```python
#!/usr/bin/env python3
import requests
import time
import sys
import os

API_BASE = "https://api.lexgo.cl/api/v1"
API_KEY = "YOUR_API_KEY"

def create_envelope():
    print("=== Step 1: Create Envelope ===")
    response = requests.post(
        f"{API_BASE}/envelopes",
        headers={"Authorization": API_KEY},
        data={
            "name": "Contract - Test",
            "recipients[0][name]": "John Doe",
            "recipients[0][email]": "john@example.com",
            "recipients[0][order]": "1"
        }
    )
    response.raise_for_status()
    envelope_id = response.json()["envelope"]["id"]
    print(f"✓ Created envelope: {envelope_id}")
    return envelope_id

def request_upload_url(envelope_id, filename):
    print("\n=== Step 2: Request Upload URL ===")
    response = requests.post(
        f"{API_BASE}/envelopes/{envelope_id}/uploads",
        headers={
            "Authorization": API_KEY,
            "Content-Type": "application/json"
        },
        json={
            "file_name": os.path.basename(filename),
            "order": 0
        }
    )
    response.raise_for_status()
    data = response.json()["upload"]
    print("✓ Got upload URL")
    print(f"✓ Upload ID: {data['id']}")
    return data["upload_url"], data["id"]

def upload_file(upload_url, pdf_path):
    print("\n=== Step 3: Upload File ===")
    with open(pdf_path, "rb") as f:
        response = requests.put(
            upload_url,
            data=f,
            headers={
                "Authorization": API_KEY,
                "Content-Type": "application/pdf"
            }
        )
    response.raise_for_status()
    print("✓ Uploaded file")

def wait_for_processing(envelope_id, upload_id):
    print("\n=== Step 4: Wait for Processing ===")
    max_attempts = 30
    for attempt in range(max_attempts):
        response = requests.get(
            f"{API_BASE}/envelopes/{envelope_id}/uploads/{upload_id}",
            headers={"Authorization": API_KEY}
        )
        response.raise_for_status()

        upload = response.json()["upload"]
        status = upload["status"]
        print(f"  Status: {status} (attempt {attempt+1}/{max_attempts})")

        if status == "COMPLETED":
            print("✓ File processed successfully!")
            print(f"✓ File ID: {upload['file_id']}")
            return True
        elif status == "FAILED":
            print(f"✗ Processing failed: {upload.get('error_message', 'Unknown error')}")
            return False

        time.sleep(2)

    print("✗ Timeout waiting for processing")
    return False

def verify_envelope(envelope_id):
    print("\n=== Step 5: Verify Envelope ===")
    response = requests.get(
        f"{API_BASE}/envelopes/{envelope_id}",
        headers={"Authorization": API_KEY}
    )
    response.raise_for_status()

    envelope = response.json()["envelope"]
    doc_count = len(envelope["documents"])
    error_count = len(envelope.get("errors", []))

    if doc_count == 0:
        print("✗ No documents attached to envelope")
        return False

    if error_count > 0:
        print(f"✗ Envelope has {error_count} error(s)")
        return False

    print(f"✓ Envelope verified: {doc_count} document(s), no errors")
    return True

def send_envelope(envelope_id):
    print("\n=== Step 6: Send Envelope ===")
    response = requests.post(
        f"{API_BASE}/envelopes/{envelope_id}/send_invitation",
        headers={"Authorization": API_KEY}
    )
    response.raise_for_status()

    if response.json()["success"]:
        print("✓ Envelope sent!")
        return True
    else:
        print("✗ Failed to send envelope")
        return False

def main():
    if len(sys.argv) != 2:
        print("Usage: python upload_contract.py <pdf_file>")
        sys.exit(1)

    pdf_path = sys.argv[1]

    try:
        envelope_id = create_envelope()
        upload_url, upload_id = request_upload_url(envelope_id, pdf_path)
        upload_file(upload_url, pdf_path)

        if not wait_for_processing(envelope_id, upload_id):
            sys.exit(1)

        if not verify_envelope(envelope_id):
            sys.exit(1)

        if not send_envelope(envelope_id):
            sys.exit(1)

        print(f"\nDone! Envelope ID: {envelope_id}")

    except requests.HTTPError as e:
        print(f"\n✗ API Error: {e.response.status_code}")
        print(e.response.json())
        sys.exit(1)
    except Exception as e:
        print(f"\n✗ Error: {e}")
        sys.exit(1)

if __name__ == "__main__":
    main()
```

Run it:

```bash
python upload_contract.py contract.pdf
```

---

## Common Issues

### "URL expired" (403 error)

**Fix:** Request a new upload URL. They expire after 1 hour.

### Status stuck in PENDING

**Fix:** Verify the upload succeeded (should return 200 with success: true).

### Status changes to FAILED

**Fix:** Check the error message and validate your PDF.

```bash
# Get error details
curl https://api.lexgo.cl/api/v1/envelopes/$ENVELOPE_ID/uploads/$UPLOAD_ID \
  -H "Authorization: YOUR_API_KEY" | jq '.upload.error_message'

# Common errors:
# - "Invalid PDF format" -> corrupted file
# - "Encrypted PDF" -> remove password
# - "File too large" -> must be <50MB
```

### Cannot create upload (422 error)

**Fix:** Another upload for this order already exists.

```bash
# Check existing uploads
curl https://api.lexgo.cl/api/v1/envelopes/$ENVELOPE_ID/uploads \
  -H "Authorization: YOUR_API_KEY" | jq '.uploads[] | select(.order == 0)'

# Wait for completion or use different order number
```

---

## Next Steps

- [File Upload API Reference](../api/uploads.md) - Complete API documentation
- [Envelopes API](../api/envelopes.md) - Standard base64 upload method
- [Quick Start Guide](../getting-started/quick-start.md) - Getting started with the API
