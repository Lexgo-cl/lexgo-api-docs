# Presigned Uploads API

Upload large PDF files (>1MB) directly to S3 instead of sending them base64-encoded in the envelope creation request.

## When to Use

Use presigned uploads when:

- File size is larger than 1MB
- You want to reduce API request time
- You want to avoid base64 encoding overhead

Use standard envelope creation (base64) when:

- File size is less than 1MB
- You want simplicity (single API call)

## How It Works

1. Create envelope (without documents)
2. Request presigned URL for each document
3. Upload PDF directly to S3 URL
4. Wait for processing (poll status endpoint OR use webhook)
5. Verify envelope has documents attached
6. Send envelope

## Upload States

| State | Description |
|-------|-------------|
| `PENDING` | URL generated, waiting for upload |
| `UPLOADED` | File received, processing |
| `PROCESSING` | Validating PDF |
| `COMPLETED` | File ready, can send envelope |
| `FAILED` | Upload or validation error |
| `EXPIRED` | URL expired (1 hour) |

## API Endpoints

### Create Presigned Upload

Generate S3 upload URL for a document.

**Endpoint:** `POST /api/v1/envelopes/:envelope_id/presigned_uploads`

**Request:**

```bash
curl -X POST https://api.lexgo.cl/api/v1/envelopes/abc-123/presigned_uploads \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "file_name": "contract.pdf",
    "order": 0,
    "custom_key": "doc1"
  }'
```

**Parameters:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file_name` | string | Yes | Filename (max 100 chars) |
| `order` | integer | Yes | Position in envelope (0, 1, 2...) |
| `custom_key` | string | No | Your tracking reference |
| `content_type` | string | No | `application/pdf` (only PDF supported) |

**Response:**

```json
{
  "success": true,
  "presigned_upload": {
    "id": "upload_abc123",
    "presigned_url": "https://lexgo-secure-staging.s3.us-east-2.amazonaws.com/...",
    "object_key": "pki/files/data/abc/123/de-/original/0_1763665515_contract.pdf",
    "bucket": "lexgo-secure-staging",
    "file_name": "contract.pdf",
    "order": 0,
    "custom_key": "doc1",
    "content_type": "application/pdf",
    "expires_at": "2025-11-20T17:05:15-03:00",
    "max_file_size": 52428800,
    "envelope_id": "abc-123",
    "status": "PENDING",
    "instructions": {
      "method": "PUT",
      "headers": {
        "Content-Type": "application/pdf"
      },
      "note": "Upload file directly to presigned_url using PUT method with Content-Type header"
    }
  },
  "request_id": "req_xyz"
}
```

**Note:** `max_file_size` is 52428800 bytes (50 MB).

**Error Responses:**

- `404` - Envelope not found
- `405` - Envelope not in CREATED status (already sent)
- `422` - Validation error (duplicate order, missing fields)

---

### Upload File to S3

Upload PDF to the presigned URL (no Authorization header needed).

**Request:**

```bash
curl -X PUT "PRESIGNED_URL_FROM_PREVIOUS_RESPONSE" \
  -H "Content-Type: application/pdf" \
  --data-binary @contract.pdf
```

**Important:**

- Use HTTP **PUT** method
- Set `Content-Type: application/pdf` header
- Send raw binary file (not base64)
- URL expires in 1 hour
- Max file size: 50MB

**Response:**

- `200` - Upload successful
- `403` - URL expired or invalid signature

---

### Check Upload Status

Check processing status of an upload.

**Endpoint:** `GET /api/v1/envelopes/:envelope_id/presigned_uploads/:id`

**Request:**

```bash
curl https://api.lexgo.cl/api/v1/envelopes/abc-123/presigned_uploads/upload_abc123 \
  -H "Authorization: YOUR_API_KEY"
```

**Response (Pending):**

```json
{
  "presigned_upload": {
    "id": "upload_abc123",
    "envelope_id": "abc-123",
    "status": "PENDING",
    "order": 0,
    "file_name": "contract.pdf",
    "custom_key": "doc1",
    "content_type": "application/pdf",
    "expires_at": "2025-11-20T17:05:15-03:00",
    "created_at": "2025-11-20T16:05:15-03:00",
    "updated_at": "2025-11-20T16:05:15-03:00"
  },
  "request_id": "req_def"
}
```

**Response (Completed):**

```json
{
  "presigned_upload": {
    "id": "upload_abc123",
    "envelope_id": "abc-123",
    "status": "COMPLETED",
    "file_id": "file_xyz789",
    "order": 0,
    "file_name": "contract.pdf",
    "custom_key": "doc1",
    "uploaded_at": "2025-11-20T19:20:15-03:00",
    "processed_at": "2025-11-20T19:20:18-03:00"
  },
  "request_id": "req_def"
}
```

**Response (Failed):**

```json
{
  "presigned_upload": {
    "id": "upload_abc123",
    "envelope_id": "abc-123",
    "status": "FAILED",
    "error_message": "Invalid PDF format",
    "order": 0,
    "file_name": "contract.pdf",
    "custom_key": "doc1"
  },
  "request_id": "req_def"
}
```

---

### List All Uploads

Get all uploads for an envelope.

**Endpoint:** `GET /api/v1/envelopes/:envelope_id/presigned_uploads`

**Request:**

```bash
# Get all uploads
curl https://api.lexgo.cl/api/v1/envelopes/abc-123/presigned_uploads \
  -H "Authorization: YOUR_API_KEY"

# Filter by status
curl https://api.lexgo.cl/api/v1/envelopes/abc-123/presigned_uploads?status=completed \
  -H "Authorization: YOUR_API_KEY"
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | Filter: `pending`, `uploaded`, `processing`, `completed`, `failed`, `expired` |

**Response:**

```json
{
  "presigned_uploads": [
    {
      "id": "upload_abc123",
      "envelope_id": "abc-123",
      "status": "COMPLETED",
      "file_id": "file_xyz789",
      "order": 0,
      "file_name": "contract.pdf",
      "custom_key": "doc1",
      "content_type": "application/pdf",
      "uploaded_at": "2025-11-20T19:20:15-03:00",
      "processed_at": "2025-11-20T19:20:18-03:00",
      "created_at": "2025-11-20T16:05:15-03:00",
      "updated_at": "2025-11-20T19:20:18-03:00"
    }
  ],
  "total": 1,
  "request_id": "req_ghi"
}
```

---

## Webhook Alternative to Polling

Instead of polling the status endpoint, you can subscribe to the `envelope.file_uploaded` webhook event.

### Subscribe to Webhook

```bash
curl -X POST https://api.lexgo.cl/api/v1/webhooks \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-app.com/webhooks/lexgo",
    "subscriptions": "envelope.file_uploaded",
    "auth_type": "header",
    "auth": "Bearer your_webhook_secret"
  }'
```

### Webhook Payload

When a file completes processing, you'll receive:

```json
{
  "id": "event_abc123",
  "event_type": "envelope.file_uploaded",
  "data": {
    "envelope_id": "abc-123",
    "presigned_upload_id": "upload_abc123",
    "file_id": "file_xyz789",
    "order": 0,
    "file_name": "contract.pdf",
    "custom_key": "doc1",
    "status": "COMPLETED",
    "uploaded_at": "2025-11-20T19:20:15-03:00",
    "processed_at": "2025-11-20T19:20:18-03:00"
  }
}
```

**Advantages:**

- No polling required
- Instant notification when file is ready
- Reduced API calls

See [Webhooks API](webhooks.md) for complete webhook documentation.

---

## Verify Documents Before Sending

After uploads complete, verify the envelope has all documents attached:

**Request:**

```bash
curl https://api.lexgo.cl/api/v1/envelopes/abc-123 \
  -H "Authorization: YOUR_API_KEY"
```

**Response:**

```json
{
  "envelope": {
    "id": "abc-123",
    "name": "Contract - John Doe",
    "status": "CREATED",
    "documents": [
      {
        "id": "file_xyz789",
        "file_name": "contract.pdf",
        "order": 0,
        "status": "READY",
        "page_count": 5,
        "file_size": 1234567
      }
    ],
    "recipients": [
      {
        "id": "recipient_001",
        "name": "John Doe",
        "email": "john@example.com",
        "status": "PENDING"
      }
    ],
    "errors": [],
    "warnings": []
  }
}
```

**Check before sending:**

- All expected documents are listed in `documents` array
- Each document has `status: "READY"`
- `errors` array is empty
- Envelope status is still `CREATED`

---

## Complete Workflow Example

```bash
# Step 1: Create envelope
ENVELOPE_JSON=$(curl -s -X POST https://api.lexgo.cl/api/v1/envelopes \
  -H "Authorization: YOUR_API_KEY" \
  -F "name=Contract - John Doe" \
  -F "recipients[0][name]=John Doe" \
  -F "recipients[0][email]=john@example.com" \
  -F "recipients[0][order]=1")

ENVELOPE_ID=$(echo "$ENVELOPE_JSON" | jq -r '.envelope.id')

# Step 2: Request presigned URL
PRESIGNED_JSON=$(curl -s -X POST \
  "https://api.lexgo.cl/api/v1/envelopes/${ENVELOPE_ID}/presigned_uploads" \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"file_name":"contract.pdf","order":0}')

PRESIGNED_URL=$(echo "$PRESIGNED_JSON" | jq -r '.presigned_upload.presigned_url')
UPLOAD_ID=$(echo "$PRESIGNED_JSON" | jq -r '.presigned_upload.id')

# Step 3: Upload to S3
curl -X PUT "$PRESIGNED_URL" \
  -H "Content-Type: application/pdf" \
  --data-binary @contract.pdf

# Step 4: Wait for processing (poll or use webhook)
for i in {1..30}; do
  STATUS_JSON=$(curl -s \
    "https://api.lexgo.cl/api/v1/envelopes/${ENVELOPE_ID}/presigned_uploads/${UPLOAD_ID}" \
    -H "Authorization: YOUR_API_KEY")

  STATUS=$(echo "$STATUS_JSON" | jq -r '.presigned_upload.status')
  echo "Status: $STATUS"

  [ "$STATUS" = "COMPLETED" ] && break
  [ "$STATUS" = "FAILED" ] && exit 1

  sleep 2
done

# Step 5: Verify envelope has documents
curl "https://api.lexgo.cl/api/v1/envelopes/${ENVELOPE_ID}" \
  -H "Authorization: YOUR_API_KEY" | jq '.envelope.documents'

# Step 6: Send envelope
curl -X POST \
  "https://api.lexgo.cl/api/v1/envelopes/${ENVELOPE_ID}/send_invitation" \
  -H "Authorization: YOUR_API_KEY"
```

---

## Error Handling

### Upload Failed (403)

**Cause:** URL expired or wrong Content-Type

**Solution:**

```bash
# Request new presigned URL (they expire after 1 hour)
curl -X POST https://api.lexgo.cl/api/v1/envelopes/$ENVELOPE_ID/presigned_uploads \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"file_name":"contract.pdf","order":0}'
```

### Status FAILED

**Cause:** Invalid PDF, corrupted file, encrypted PDF

**Solution:**

```bash
# Get error details
curl https://api.lexgo.cl/api/v1/envelopes/$ENVELOPE_ID/presigned_uploads/$UPLOAD_ID \
  -H "Authorization: YOUR_API_KEY" | jq '.presigned_upload.error_message'

# Common errors:
# - "Invalid PDF format" -> check file is valid PDF
# - "Encrypted PDF" -> remove password protection
# - "File too large" -> must be <50MB
```

### Cannot Create Upload (422)

**Cause:** Another upload for this order already exists

**Solution:**

```bash
# List existing uploads
curl https://api.lexgo.cl/api/v1/envelopes/$ENVELOPE_ID/presigned_uploads \
  -H "Authorization: YOUR_API_KEY" | \
  jq '.presigned_uploads[] | select(.order == 0)'

# Wait for completion or use different order number
```

---

## Best Practices

### 1. Use Webhooks Instead of Polling

```bash
# Set up webhook once
curl -X POST https://api.lexgo.cl/api/v1/webhooks \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-app.com/webhooks/lexgo",
    "subscriptions": "envelope.file_uploaded"
  }'

# Your webhook endpoint receives notification when file is ready
# No need to poll status endpoint
```

### 2. Always Verify Envelope Before Sending

```bash
# Check documents are attached
ENVELOPE=$(curl -s https://api.lexgo.cl/api/v1/envelopes/$ENVELOPE_ID \
  -H "Authorization: YOUR_API_KEY")

# Verify no errors
ERRORS=$(echo "$ENVELOPE" | jq '.envelope.errors')
if [ "$ERRORS" != "[]" ]; then
  echo "Envelope has errors, cannot send"
  exit 1
fi

# Verify expected document count
DOC_COUNT=$(echo "$ENVELOPE" | jq '.envelope.documents | length')
if [ "$DOC_COUNT" != "1" ]; then
  echo "Expected 1 document, found $DOC_COUNT"
  exit 1
fi

# Safe to send
curl -X POST https://api.lexgo.cl/api/v1/envelopes/$ENVELOPE_ID/send_invitation \
  -H "Authorization: YOUR_API_KEY"
```

### 3. Handle Upload Retries

```bash
# Retry S3 upload on failure
MAX_RETRIES=3
for attempt in $(seq 1 $MAX_RETRIES); do
  HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" \
    -X PUT "$PRESIGNED_URL" \
    -H "Content-Type: application/pdf" \
    --data-binary @contract.pdf)

  if [ "$HTTP_CODE" = "200" ]; then
    break
  fi

  if [ $attempt -eq $MAX_RETRIES ]; then
    echo "Upload failed after $MAX_RETRIES attempts"
    exit 1
  fi

  sleep $((2 ** attempt))  # Exponential backoff
done
```

### 4. Validate PDFs Before Upload

```bash
# Check if file is a valid PDF
if ! file contract.pdf | grep -q "PDF"; then
  echo "Error: Not a valid PDF file"
  exit 1
fi

# Check file size (<50MB)
FILE_SIZE=$(stat -f%z contract.pdf)
if [ $FILE_SIZE -gt 52428800 ]; then
  echo "Error: File too large (max 50MB)"
  exit 1
fi
```

---

## Related Documentation

- [Presigned Uploads Guide](../guides/presigned-uploads-guide.md) - Complete implementations in Python, Ruby, Bash
- [Envelopes API](envelopes.md) - Standard base64 upload method
- [Webhooks API](webhooks.md) - Real-time event notifications
- [Quick Start](../getting-started/quick-start.md) - Getting started

---

## FAQ

**Q: When should I use presigned uploads vs base64?**

A: Use presigned uploads for files >1MB. Use base64 for smaller files.

**Q: Should I poll or use webhooks?**

A: Use webhooks for production. Polling is simpler for testing but less efficient.

**Q: How long does processing take?**

A: Typically 2-10 seconds from S3 upload to COMPLETED status.

**Q: Can I upload multiple files?**

A: Yes, request a presigned URL for each file with different order values.

**Q: What if URL expires?**

A: Request a new presigned URL. They expire after 1 hour.
