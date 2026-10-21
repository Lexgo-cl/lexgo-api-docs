# File Upload API

Upload PDF files to envelopes using the Upload API.

!!! danger "Legacy streaming uploads are retired and not supported"
    The streaming upload routes (`POST`/`GET /v1/envelopes/{id}/uploads`, `GET .../uploads/{id}` and the `PUT` to the returned `upload_url`) are **legacy**. They were retired by an owner decision (issue #1207), which retains presigned URLs. The routes still exist in the source and are documented below only with that status. On beta, `POST .../uploads` returned an `upload_url` on `https://api-staging.lexgo.cl`, a host that does not resolve, and the gateway Lambda's own API base URL is configured with the same host (CDK `service-api-layer`); this is retirement evidence, not a request to fix it. Do not guess another prefix and do not call the gateway directly. **Use [presigned uploads](#presigned-uploads).**

## When to Use

Use the upload API when:

- File size is larger than 1MB (base64 documents in envelope create/update are limited to 5 MB)
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

*Legacy, retired per issue #1207, not supported. Use [presigned uploads](#presigned-uploads).*

Generate an upload URL for a document.

**Endpoint:** `POST /v1/envelopes/:envelope_id/uploads`

**Request:**

```bash
curl -X POST https://api.lexgo.cl/v1/envelopes/abc-123/uploads \
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
    "max_file_size": 10485760,
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

**Note:** `max_file_size` is the legacy value 10485760 bytes (10 MB); it is not a promise that a 10 MB upload completes. Use [presigned uploads](#presigned-uploads) (up to 50 MB).

**Error Responses:**

- `400` - Missing `file_name` or `order`, `file_name` over 100 characters, or `content_type` other than `application/pdf`
- `404` - Envelope not found
- `405` - Envelope not in CREATED status (already sent); the body also has `current_status`
- `422` - Another active upload already exists for this `order`

---

### Upload File

*Legacy, retired per issue #1207, not supported. Use [presigned uploads](#presigned-uploads).*

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
- Legacy max file size: 10 MB (larger requests are rejected with `413`)

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

*Legacy, retired per issue #1207, not supported. For presigned uploads use `GET .../presigned_uploads/{id}`.*

Check processing status of an upload.

**Endpoint:** `GET /v1/envelopes/:envelope_id/uploads/:id`

**Request:**

```bash
curl https://api.lexgo.cl/v1/envelopes/abc-123/uploads/upload_abc123 \
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

*Legacy, retired per issue #1207, not supported. The same records are returned by `GET .../presigned_uploads`.*

Get all uploads for an envelope.

**Endpoint:** `GET /v1/envelopes/:envelope_id/uploads`

**Request:**

```bash
# Get all uploads
curl https://api.lexgo.cl/v1/envelopes/abc-123/uploads \
  -H "Authorization: YOUR_API_KEY"

# Filter by status
curl "https://api.lexgo.cl/v1/envelopes/abc-123/uploads?status=completed" \
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

## Presigned Uploads

Presigned uploads send the PDF straight to storage with a presigned URL, and accept files up to 50 MB. They are the supported upload flow. The workflow, states and constraints are the same as above; only the endpoints and the upload step differ.

**Endpoints:**

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/v1/envelopes/:envelope_id/presigned_uploads` | Create a presigned upload |
| `GET` | `/v1/envelopes/:envelope_id/presigned_uploads` | List presigned uploads (optional `?status=`: `pending`, `uploaded`, `processing`, `completed`, `failed`, `expired`) |
| `GET` | `/v1/envelopes/:envelope_id/presigned_uploads/:id` | Check one presigned upload |

`POST` takes the same parameters and validation as [Create Upload](#create-upload) (`file_name` and `order` required, `custom_key` and `content_type` optional, PDF only).

```bash
curl -X POST https://api.lexgo.cl/v1/envelopes/abc-123/presigned_uploads \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"file_name": "contract.pdf", "order": 0, "custom_key": "doc1"}'
```

**Response (200):**

```json
{
  "success": true,
  "presigned_upload": {
    "id": "550e8400-e29b-41d4-a716-446655440002",
    "presigned_url": "https://<storage-host>/...?X-Amz-Signature=...",
    "file_name": "contract.pdf",
    "order": 0,
    "custom_key": "doc1",
    "content_type": "application/pdf",
    "expires_at": "2025-11-20T17:05:15Z",
    "max_file_size": 52428800,
    "envelope_id": "abc-123",
    "status": "PENDING",
    "instructions": {
      "method": "PUT",
      "headers": { "Content-Type": "application/pdf" },
      "note": "Upload file directly to presigned_url using PUT method with Content-Type header"
    }
  },
  "request_id": "req_xyz"
}
```

**Upload the file** with a `PUT` of the raw PDF bytes to `presigned_url`. Send the same `Content-Type` header as in `instructions.headers`, and do **not** send your Lexgo `Authorization` header; the signature is in the URL.

```bash
curl -X PUT "{presigned_url}" \
  -H "Content-Type: application/pdf" \
  --data-binary @contract.pdf
```

The URL expires 1 hour after creation (`expires_at`). Processing starts after the file arrives; poll `GET .../presigned_uploads/:id` or subscribe to `envelope.file_uploaded`. The status values are the [Upload States](#upload-states). The status object has `id`, `envelope_id`, `status`, `order`, `file_name`, `custom_key`, `content_type`, `upload_url`, `file_id`, `error_message`, `expires_at`, `uploaded_at`, `processed_at`, `created_at` and `updated_at` (absent values are omitted); the list response is `{"presigned_uploads": [...], "total": n}`. The `upload_url` field in these status objects is the `/uploads` streaming URL and is not used for presigned uploads.

**Error Responses:** the same as [Create Upload](#create-upload) (`400`, `404`, `405`, `422`), plus `500` (`Failed to generate presigned URL`). An unknown presigned upload ID returns `404` (`Presigned upload not found`).

!!! note "Staging host"
    In staging environments the `upload_url` returned by `/uploads` and the status endpoints is built on `https://api-staging.lexgo.cl`, while the staging API host is `https://api-beta.lexgo.cl`. The `api-staging` host does not resolve on beta (retirement evidence; see the notice at the top of this page). Presigned uploads do not use this host.

---

## Webhook Alternative to Polling

Subscribe to `envelope.file_uploaded` for real-time notifications instead of polling:

```bash
curl -X POST https://api.lexgo.cl/v1/webhooks \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://your-app.com/webhooks/lexgo",
    "subscriptions": "envelope.file_uploaded"
  }'
```

Your webhook will receive notification when file is ready (use polling to reconcile missed notifications).

See [Webhooks API](webhooks.md) for complete documentation.

---

## Complete Workflow Example (legacy streaming, retired)

*Legacy, retired per issue #1207, not supported. Kept for reference only; follow the presigned workflow instead.*

```bash
# Step 1: Create envelope
ENVELOPE_JSON=$(curl -s -X POST https://api.lexgo.cl/v1/envelopes \
  -H "Authorization: YOUR_API_KEY" \
  -F "name=Contract - John Doe" \
  -F "recipients[0][name]=John Doe" \
  -F "recipients[0][email]=john@example.com" \
  -F "recipients[0][order]=1")

ENVELOPE_ID=$(echo "$ENVELOPE_JSON" | jq -r '.envelope.id')

# Step 2: Request upload URL
UPLOAD_JSON=$(curl -s -X POST \
  "https://api.lexgo.cl/v1/envelopes/${ENVELOPE_ID}/uploads" \
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
    "https://api.lexgo.cl/v1/envelopes/${ENVELOPE_ID}/uploads/${UPLOAD_ID}" \
    -H "Authorization: YOUR_API_KEY")

  STATUS=$(echo "$STATUS_JSON" | jq -r '.upload.status')
  echo "Status: $STATUS"

  [ "$STATUS" = "COMPLETED" ] && break
  [ "$STATUS" = "FAILED" ] && exit 1

  sleep 2
done

# Step 5: Add the required document/recipient placement, then verify
curl -X PUT "https://api.lexgo.cl/v1/envelopes/${ENVELOPE_ID}" \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"placements":{"signature":{"document_key":"0","recipient_key":"0","type":"SIGNATURE"}}}'

# Verify the envelope has documents and no errors
curl "https://api.lexgo.cl/v1/envelopes/${ENVELOPE_ID}" \
  -H "Authorization: YOUR_API_KEY" | jq '.envelope.documents'

# Step 6: Send envelope (HTTP 202/SENDING; reconcile subsequent status)
curl -X PUT \
  "https://api.lexgo.cl/v1/envelopes/${ENVELOPE_ID}/send" \
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
curl https://api.lexgo.cl/v1/envelopes/$ENVELOPE_ID/uploads/$UPLOAD_ID \
  -H "Authorization: YOUR_API_KEY" | jq '.upload.error_message'
```

### Cannot Create Upload (422)

**Cause:** Another upload for this order already exists

**Solution:** Wait for completion or use different order number.

---

## Constraints

- **File type**: PDF only (`application/pdf`)
- **Max file size**: 50 MB for [presigned uploads](#presigned-uploads) (legacy `/uploads`: 10 MB, retired) (base64 in envelope create/update: 5 MB)
- **Max filename**: 100 characters
- **URL expiration**: 1 hour
- **Single use**: Each upload URL can only be used once
- **One active per order**: Only one pending/uploaded/processing upload per order position

---

## Related Documentation

- [Uploads Guide](../guides/uploads-guide.md) - Complete implementations in Python, Ruby, Bash
- [Envelopes API](envelopes.md) - Standard base64 upload method
- [Webhooks API](webhooks.md) - Real-time event notifications
