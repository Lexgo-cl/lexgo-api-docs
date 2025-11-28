# File Upload API

Upload PDF files to envelopes using the Upload API.

## When to Use

Use the upload API when:

- File size is larger than 1MB
- You want to reduce API request time
- You want to avoid base64 encoding overhead

Use standard envelope creation (base64) when:

- File size is less than 1MB
- You want simplicity (single API call)

## How It Works

1. Create envelope (without documents)
2. Request upload URL for each document
3. Upload PDF to the upload URL
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

### Create Upload

Generate an upload URL for a document.

**Endpoint:** `POST /api/v1/envelopes/:envelope_id/uploads`

**Request:**

```bash
curl -X POST https://api.lexgo.cl/api/v1/envelopes/abc-123/uploads \
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
  "upload": {
    "id": "550e8400-e29b-41d4-a716-446655440001",
    "upload_url": "https://api.lexgo.cl/v1/uploads/550e8400.../stream",
    "file_name": "contract.pdf",
    "order": 0,
    "custom_key": "doc1",
    "expires_at": "2025-11-20T17:05:15Z",
    "max_file_size": 52428800,
    "envelope_id": "abc-123",
    "status": "PENDING",
    "instructions": {
      "method": "PUT",
      "headers": { "Content-Type": "application/pdf" },
      "note": "Upload file to upload_url using PUT method. Request body should contain the binary file content."
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

### Upload File

Upload PDF to the upload URL.

**Request:**

```bash
curl -X PUT "{upload_url}" \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/pdf" \
  --data-binary @contract.pdf
```

**Important:**

- Use HTTP **PUT** method
- Include `Authorization` header with your API key
- Set `Content-Type: application/pdf` header
- Send raw binary file (not base64)
- URL expires in 1 hour
- Max file size: 50MB

**Response:**

```json
{
  "success": true,
  "upload_id": "550e8400-e29b-41d4-a716-446655440001",
  "status": "UPLOADED",
  "message": "File uploaded successfully"
}
```

---

### Check Upload Status

Check processing status of an upload.

**Endpoint:** `GET /api/v1/envelopes/:envelope_id/uploads/:id`

**Request:**

```bash
curl https://api.lexgo.cl/api/v1/envelopes/abc-123/uploads/upload_abc123 \
  -H "Authorization: YOUR_API_KEY"
```

**Response (Completed):**

```json
{
  "upload": {
    "id": "upload_abc123",
    "envelope_id": "abc-123",
    "status": "COMPLETED",
    "file_id": "file_xyz789",
    "order": 0,
    "file_name": "contract.pdf",
    "custom_key": "doc1",
    "uploaded_at": "2025-11-20T19:20:15Z",
    "processed_at": "2025-11-20T19:20:18Z"
  },
  "request_id": "req_def"
}
```

---

### List All Uploads

Get all uploads for an envelope.

**Endpoint:** `GET /api/v1/envelopes/:envelope_id/uploads`

**Request:**

```bash
# Get all uploads
curl https://api.lexgo.cl/api/v1/envelopes/abc-123/uploads \
  -H "Authorization: YOUR_API_KEY"

# Filter by status
curl "https://api.lexgo.cl/api/v1/envelopes/abc-123/uploads?status=completed" \
  -H "Authorization: YOUR_API_KEY"
```

**Response:**

```json
{
  "uploads": [
    {
      "id": "upload_abc123",
      "envelope_id": "abc-123",
      "status": "COMPLETED",
      "file_id": "file_xyz789",
      "order": 0,
      "file_name": "contract.pdf"
    }
  ],
  "total": 1,
  "request_id": "req_ghi"
}
```

---

## Webhook Alternative to Polling

Subscribe to `envelope.file_uploaded` for real-time notifications instead of polling:

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

See [Webhooks API](webhooks.md) for complete documentation.

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

# Step 2: Request upload URL
UPLOAD_JSON=$(curl -s -X POST \
  "https://api.lexgo.cl/api/v1/envelopes/${ENVELOPE_ID}/uploads" \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"file_name":"contract.pdf","order":0}')

UPLOAD_URL=$(echo "$UPLOAD_JSON" | jq -r '.upload.upload_url')
UPLOAD_ID=$(echo "$UPLOAD_JSON" | jq -r '.upload.id')

# Step 3: Upload file
curl -X PUT "$UPLOAD_URL" \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/pdf" \
  --data-binary @contract.pdf

# Step 4: Wait for processing
for i in {1..30}; do
  STATUS_JSON=$(curl -s \
    "https://api.lexgo.cl/api/v1/envelopes/${ENVELOPE_ID}/uploads/${UPLOAD_ID}" \
    -H "Authorization: YOUR_API_KEY")

  STATUS=$(echo "$STATUS_JSON" | jq -r '.upload.status')
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

**Solution:** Request new upload URL (they expire after 1 hour).

### Status FAILED

**Cause:** Invalid PDF, corrupted file, encrypted PDF

**Solution:** Check error_message in upload status response:

```bash
curl https://api.lexgo.cl/api/v1/envelopes/$ENVELOPE_ID/uploads/$UPLOAD_ID \
  -H "Authorization: YOUR_API_KEY" | jq '.upload.error_message'
```

### Cannot Create Upload (422)

**Cause:** Another upload for this order already exists

**Solution:** Wait for completion or use different order number.

---

## Constraints

- **File type**: PDF only (`application/pdf`)
- **Max file size**: 50 MB
- **Max filename**: 100 characters
- **URL expiration**: 1 hour
- **Single use**: Each upload URL can only be used once
- **One active per order**: Only one pending/uploaded/processing upload per order position

---

## Related Documentation

- [Uploads Guide](../guides/uploads-guide.md) - Complete implementations in Python, Ruby, Bash
- [Envelopes API](envelopes.md) - Standard base64 upload method
- [Webhooks API](webhooks.md) - Real-time event notifications
