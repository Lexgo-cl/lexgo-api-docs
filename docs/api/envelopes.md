# Envelopes API

Envelopes are the core resource in LexgoSign. An envelope contains documents and recipients (signers), and manages the complete signature workflow from creation to completion.

## Overview

An envelope represents a signature request with one or more documents and recipients. Envelopes progress through several states:

```
CREATED → IN_PROGRESS → SUCCESS
   ↓
 VOIDED
```

### Envelope States

| State | Description |
|-------|-------------|
| `CREATED` | Envelope created but not sent to recipients |
| `IN_PROGRESS` | Envelope sent, awaiting signatures |
| `SUCCESS` | All recipients have signed |
| `VOIDED` | Envelope cancelled before completion |

## Endpoints

### Create Envelope

Create a new envelope with documents and recipients.

**Endpoint:** `POST /api/v1/envelopes`

=== "Request"
    ```http
    POST /api/v1/envelopes HTTP/1.1
    Host: api.lexgo.cl
    Authorization: YOUR_API_KEY
    Content-Type: multipart/form-data

    name=Employment Contract - John Doe
    documents[0][base64]=JVBERi0xLjQKJeLj...
    documents[0][name]=employment-contract.pdf
    recipients[0][name]=John Doe
    recipients[0][email]=john.doe@example.com
    recipients[0][order]=1
    ```

=== "Response (200 OK)"
    ```json
    {
      "envelope": {
        "id": "abc-123-def-456",
        "name": "Employment Contract - John Doe",
        "status": "CREATED",
        "created_at": "2024-01-15T10:30:00Z",
        "envelope_signers": [
          {
            "id": "signer-001",
            "name": "John Doe",
            "email": "john.doe@example.com",
            "order": 1,
            "status": "PENDING"
          }
        ]
      },
      "request_id": "req-abc123"
    }
    ```

**Request Parameters:**

!!! note "Parameter Format"
    This API uses **multipart/form-data** with bracket notation for nested parameters. You can use numeric indices like `recipients[0]` or custom keys like `recipients[john]`.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | No | Envelope name/title (defaults to first document name) |
| `documents[key][...]` | object | Yes | Document parameters (see below) |
| `recipients[key][...]` | object | Yes | Recipient parameters (see below) |
| `placements[index][...]` | object | No | Placement parameters (see below) |
| `settings[...]` | object | No | Override default settings (see [Settings API](settings.md)) |

**Document Parameters:**

Use `documents[key][field]` where `key` can be a numeric index (0, 1, 2...) or custom identifier.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `documents[key][base64]` | string | Yes | Base64-encoded PDF content |
| `documents[key][name]` | string | No | File name (defaults to random ID) |
| `documents[key][type]` | string | No | Document type: `SIGNABLE` (default), `ATTACHMENT` |
| `documents[key][order]` | integer | No | Display order (defaults to key alphabetical order) |

**Recipient Parameters:**

Use `recipients[key][field]` where `key` can be a numeric index or custom identifier (referenced in placements).

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `recipients[key][email]` | string | Yes | Recipient email address |
| `recipients[key][name]` | string | Yes | Recipient full name |
| `recipients[key][order]` | integer | No | Signing order (defaults to key alphabetical order) |
| `recipients[key][phone]` | string | No* | Phone number (* required if phone validation enabled) |
| `recipients[key][tax_id]` | string | No | Tax ID / National ID (RUT, SSN, etc.) |
| `recipients[key][tax_label]` | string | No | Label for tax_id field (e.g., "RUT", "SSN") |
| `recipients[key][rep_id]` | string | No | Legal representative ID (company name) |
| `recipients[key][rep_label]` | string | No | Label for rep_id field (defaults to "Rep.") |

**Placement Parameters (Optional):**

Placements define where signature fields appear on documents. If not provided, recipients can sign anywhere (extra pages added at the end).

Use `placements[index][field]` where `index` is a numeric index (0, 1, 2...).

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `placements[index][document_key]` | string | Yes | Document key from `documents[key]` |
| `placements[index][recipient_key]` | string | Yes | Recipient key from `recipients[key]` |
| `placements[index][type]` | string | No | Field type: `SIGNATURE` (default), `APPROVAL`, `WITNESS` |
| `placements[index][order]` | integer | No | Display order (defaults to index) |
| `placements[index][coordinates][top]` | decimal | No* | Y-coordinate from top of page (pixels) |
| `placements[index][coordinates][left]` | decimal | No* | X-coordinate from left of page (pixels) |
| `placements[index][coordinates][page]` | integer | No* | Page number (0-indexed, 0 = first page) |
| `placements[index][token]` | string | No* | Token to position signature (alternative to coordinates) |

!!! note "Positioning"
    You must provide **either** coordinates (top/left/page) **or** a token. If neither is provided, signatures will be added on extra pages at the end.

**Example with Placements:**

```http
POST /api/v1/envelopes HTTP/1.1
Host: api.lexgo.cl
Authorization: YOUR_API_KEY
Content-Type: multipart/form-data

name=Employment Contract - John Doe
documents[0][base64]=JVBERi0xLjQK...
documents[0][name]=contract.pdf
recipients[john][name]=John Doe
recipients[john][email]=john.doe@example.com
recipients[john][tax_id]=12.345.678-9
recipients[john][tax_label]=RUT
recipients[john][order]=1
placements[0][document_key]=0
placements[0][recipient_key]=john
placements[0][type]=SIGNATURE
placements[0][coordinates][top]=650
placements[0][coordinates][left]=100
placements[0][coordinates][page]=0
placements[0][order]=0
```

**Alternative: Using Token Instead of Coordinates**

```http
placements[0][document_key]=0
placements[0][recipient_key]=john
placements[0][token]={custom_signature_token}
```

---

### Get Envelope

Retrieve details of a specific envelope.

**Endpoint:** `GET /api/v1/envelopes/:id`

=== "Request"
    ```bash
    curl https://api.lexgo.cl/api/v1/envelopes/abc-123-def-456 \
      -H "Authorization: YOUR_API_KEY"
    ```

=== "Response (200 OK)"
    ```json
    {
      "envelope": {
        "id": "abc-123-def-456",
        "name": "Employment Contract - John Doe",
        "status": "IN_PROGRESS",
        "created_at": "2024-01-15T10:30:00Z",
        "sent_at": "2024-01-15T10:35:00Z",
        "envelope_signers": [
          {
            "id": "signer-001",
            "name": "John Doe",
            "email": "john.doe@example.com",
            "order": 1,
            "status": "SIGNED",
            "signed_at": "2024-01-15T11:00:00Z"
          }
        ]
      },
      "request_id": "req-def456"
    }
    ```

=== "Response (404 Not Found)"
    ```json
    {
      "error": "Envelope not found",
      "request_id": "req-ghi789"
    }
    ```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Unique envelope identifier |
| `name` | string | Envelope name |
| `status` | string | Current status (CREATED, IN_PROGRESS, SUCCESS, VOIDED) |
| `created_at` | datetime | When envelope was created |
| `sent_at` | datetime | When envelope was sent to recipients |
| `completed_at` | datetime | When all signatures completed |
| `envelope_signers` | array | List of recipients with their status |

---

### Update Envelope

Update an existing envelope before it's sent to recipients.

**Endpoint:** `PUT /api/v1/envelopes/:id`

!!! info "Update Restrictions"
    Envelopes can only be updated while in `CREATED` or `ERROR` status. Once sent (`IN_PROGRESS`), updates are no longer allowed.

!!! tip "Error Recovery"
    If an envelope is in `ERROR` status and you fix the validation errors, the status will automatically transition back to `CREATED`.

=== "Request"
    ```http
    PUT /api/v1/envelopes/abc-123 HTTP/1.1
    Host: api.lexgo.cl
    Authorization: YOUR_API_KEY
    Content-Type: multipart/form-data

    name=Updated Employment Contract
    documents[0][base64]=JVBERi0xLjQKJeLj...
    documents[0][name]=updated-contract.pdf
    recipients[0][name]=Jane Smith
    recipients[0][email]=jane.smith@example.com
    placements[0][document_key]=0
    placements[0][recipient_key]=0
    placements[0][coordinates][page]=0
    placements[0][coordinates][top]=100
    placements[0][coordinates][left]=100
    ```

=== "Response (200 OK)"
    ```json
    {
      "envelope": {
        "id": "abc-123-def-456",
        "name": "Updated Employment Contract",
        "status": "CREATED",
        "errors": [],
        "updated_at": "2024-01-15T11:30:00Z"
      },
      "request_id": "req-update123"
    }
    ```

=== "Response (405 Method Not Allowed)"
    ```json
    {
      "error": "Envelope can only be updated when in CREATED status",
      "request_id": "req-update456"
    }
    ```

=== "Response (422 Unprocessable Content)"
    ```json
    {
      "error": "Update failed",
      "message": "Invalid document format",
      "request_id": "req-update789"
    }
    ```

**Request Parameters:**

All parameters are optional. Only include the components you want to update.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | No | Update envelope name |
| `documents[key][...]` | object | No | Replace all documents (deletion by omission) |
| `recipients[key][...]` | object | No | Replace all recipients (deletion by omission) |
| `placements[key][...]` | object | No | Replace all placements (deletion by omission) |
| `settings[...]` | object | No | Update settings (see [Settings API](settings.md)) |

!!! warning "Replacement Behavior"
    When you update a component (documents, recipients, or placements), **all existing items are replaced**. To delete an item, simply omit it from the update. To keep existing items, include them in the update.

**Update Scenarios:**

| Scenario | Parameters | Result |
|----------|-----------|--------|
| Update name only | `name=New Name` | Name changes, everything else unchanged |
| Replace documents | `documents[0][...]` | All old documents deleted, new ones added |
| Update settings | `settings[...]` | Settings merged with defaults |
| Fix validation errors | Any params that fix errors | Status transitions from `ERROR` → `CREATED` |

**Status Transitions:**

- `CREATED` → `CREATED`: Normal update
- `ERROR` → `CREATED`: Automatic when validation errors are fixed
- `ERROR` → `ERROR`: When validation errors still exist
- `IN_PROGRESS`, `SUCCESS`, `VOIDED`: Updates not allowed (returns 405)

---

### Send Envelope

Send the envelope to recipients for signing.

**Endpoint:** `POST /api/v1/envelopes/:id/send_invitation`

!!! warning "One-Time Operation"
    An envelope can only be sent once. After sending, the envelope transitions from `CREATED` to `IN_PROGRESS`.

=== "Request"
    ```bash
    curl -X POST https://api.lexgo.cl/api/v1/envelopes/abc-123/send_invitation \
      -H "Authorization: YOUR_API_KEY"
    ```

=== "Response (200 OK)"
    ```json
    {
      "success": true,
      "request_id": "req-jkl012"
    }
    ```

=== "Response (405 Method Not Allowed)"
    ```json
    {
      "error": "Envelope not in CREATED status",
      "request_id": "req-mno345"
    }
    ```

**Behavior:**
- Sends invitation emails to all recipients
- Changes envelope status from `CREATED` to `IN_PROGRESS`
- Generates unique access tokens for each recipient
- Returns immediately (email sending happens asynchronously)

---

### Void Envelope

Cancel an envelope before completion.

**Endpoint:** `PUT /api/v1/envelopes/:id/void`

!!! warning "Irreversible"
    Voiding an envelope cannot be undone. Recipients will no longer be able to access or sign the documents.

=== "Request"
    ```bash
    curl -X PUT https://api.lexgo.cl/api/v1/envelopes/abc-123/void \
      -H "Authorization: YOUR_API_KEY"
    ```

=== "Response (200 OK)"
    ```json
    {
      "success": true,
      "envelope": {
        "id": "abc-123",
        "status": "VOIDED"
      },
      "request_id": "req-pqr678"
    }
    ```

=== "Response (405 Method Not Allowed)"
    ```json
    {
      "error": "Cannot void completed envelope",
      "request_id": "req-stu901"
    }
    ```

**Behavior:**
- Changes envelope status to `VOIDED`
- Invalidates all recipient access tokens
- Cannot void envelopes that are already `SUCCESS`

---

### Get Evidence Sheet

Retrieve the evidence sheet URL for a completed envelope.

**Endpoint:** `GET /api/v1/envelopes/:id/evidence`

!!! success "Optimized Performance"
    This endpoint uses intelligent async caching for **20-50x faster** response times on subsequent requests.

=== "Request"
    ```bash
    curl https://api.lexgo.cl/api/v1/envelopes/abc-123/evidence \
      -H "Authorization: YOUR_API_KEY"
    ```

=== "Response (200 OK)"
    ```json
    {
      "success": true,
      "evidence_document_url": "https://s3.amazonaws.com/.../evidence_sheet.pdf",
      "request_id": "req-vwx234"
    }
    ```

=== "Response (405 Method Not Allowed)"
    ```json
    {
      "error": "Envelope not in SUCCESS status",
      "request_id": "req-yza567"
    }
    ```

**Requirements:**
- Envelope must be in `SUCCESS` status
- All recipients must have completed signing

**Performance:**
- First request: 1-3 seconds (generates PDF)
- Subsequent requests: <100ms (returns cached URL)
- Automatic background updates when timeline changes

See the [Evidence Sheet Guide](../guides/evidence-sheet.md) for detailed documentation.

---

## Code Examples

### Complete Workflow

=== "Python"
    ```python
    import requests
    import base64
    import time

    API_BASE = "https://api.lexgo.cl/api/v1"
    API_KEY = "your_api_key_here"
    headers = {
        "Authorization": API_KEY
    }

    # 1. Create envelope
    with open('contract.pdf', 'rb') as f:
        pdf_content = base64.b64encode(f.read()).decode('utf-8')

    # Using form-data with bracket notation
    form_data = {
        "name": "Employment Contract - John Doe",
        "documents[0][base64]": pdf_content,
        "documents[0][name]": "employment-contract.pdf",
        "recipients[0][name]": "John Doe",
        "recipients[0][email]": "john.doe@example.com",
        "recipients[0][order]": "1"
    }

    create_response = requests.post(
        f"{API_BASE}/envelopes",
        headers=headers,
        data=form_data
    )

    envelope = create_response.json()["envelope"]
    envelope_id = envelope["id"]
    print(f"✓ Created envelope: {envelope_id}")

    # 2. Send to recipients
    send_response = requests.post(
        f"{API_BASE}/envelopes/{envelope_id}/send_invitation",
        headers=headers
    )

    if send_response.json()["success"]:
        print("✓ Envelope sent to recipients")

    # 3. Check status
    status_response = requests.get(
        f"{API_BASE}/envelopes/{envelope_id}",
        headers=headers
    )

    status = status_response.json()["envelope"]["status"]
    print(f"✓ Current status: {status}")

    # 4. Wait for completion and get evidence (optional)
    # In production, use webhooks instead of polling
    while status not in ["SUCCESS", "VOIDED"]:
        time.sleep(30)  # Poll every 30 seconds
        status_response = requests.get(
            f"{API_BASE}/envelopes/{envelope_id}",
            headers=headers
        )
        status = status_response.json()["envelope"]["status"]
        print(f"Status: {status}")

    if status == "SUCCESS":
        evidence_response = requests.get(
            f"{API_BASE}/envelopes/{envelope_id}/evidence",
            headers=headers
        )
        evidence_url = evidence_response.json()["evidence_document_url"]
        print(f"✓ Evidence available: {evidence_url}")
    ```

=== "JavaScript"
    ```javascript
    const fetch = require('node-fetch');
    const FormData = require('form-data');
    const fs = require('fs');

    const API_BASE = 'https://api.lexgo.cl/api/v1';
    const API_KEY = 'your_api_key_here';

    async function createAndSendEnvelope() {
      // 1. Create envelope
      const pdfBuffer = fs.readFileSync('contract.pdf');
      const pdfBase64 = pdfBuffer.toString('base64');

      // Using form-data with bracket notation
      const formData = new FormData();
      formData.append('name', 'Employment Contract - John Doe');
      formData.append('documents[0][base64]', pdfBase64);
      formData.append('documents[0][name]', 'employment-contract.pdf');
      formData.append('recipients[0][name]', 'John Doe');
      formData.append('recipients[0][email]', 'john.doe@example.com');
      formData.append('recipients[0][order]', '1');

      const createResponse = await fetch(`${API_BASE}/envelopes`, {
        method: 'POST',
        headers: {
          'Authorization': API_KEY
        },
        body: formData
      });

      const { envelope } = await createResponse.json();
      const envelopeId = envelope.id;
      console.log(`✓ Created envelope: ${envelopeId}`);

      // 2. Send to recipients
      const sendResponse = await fetch(
        `${API_BASE}/envelopes/${envelopeId}/send_invitation`,
        {
          method: 'POST',
          headers: {
            'Authorization': API_KEY
          }
        }
      );

      const { success } = await sendResponse.json();
      if (success) {
        console.log('✓ Envelope sent to recipients');
      }

      return envelopeId;
    }

    createAndSendEnvelope();
    ```

=== "Ruby"
    ```ruby
    require 'net/http'
    require 'base64'

    API_BASE = 'https://api.lexgo.cl/api/v1'
    API_KEY = 'your_api_key_here'

    # 1. Create envelope
    pdf_content = Base64.strict_encode64(File.read('contract.pdf'))

    # Using form-data with bracket notation
    form_data = [
      ['name', 'Employment Contract - John Doe'],
      ['documents[0][base64]', pdf_content],
      ['documents[0][name]', 'employment-contract.pdf'],
      ['recipients[0][name]', 'John Doe'],
      ['recipients[0][email]', 'john.doe@example.com'],
      ['recipients[0][order]', '1']
    ]

    uri = URI("#{API_BASE}/envelopes")
    request = Net::HTTP::Post.new(uri)
    request['Authorization'] = API_KEY
    request.set_form(form_data, 'multipart/form-data')

    response = Net::HTTP.start(uri.hostname, uri.port, use_ssl: true) do |http|
      http.request(request)
    end

    envelope = JSON.parse(response.body)['envelope']
    envelope_id = envelope['id']
    puts "✓ Created envelope: #{envelope_id}"

    # 2. Send to recipients
    uri = URI("#{API_BASE}/envelopes/#{envelope_id}/send_invitation")
    request = Net::HTTP::Post.new(uri)
    request['Authorization'] = API_KEY

    response = Net::HTTP.start(uri.hostname, uri.port, use_ssl: true) do |http|
      http.request(request)
    end

    result = JSON.parse(response.body)
    puts '✓ Envelope sent to recipients' if result['success']
    ```

---

## Best Practices

### 1. Use Webhooks for Status Updates

Don't poll the API to check envelope status. Use webhooks instead:

```python
# BAD: Polling
while True:
    response = requests.get(f"{API_BASE}/envelopes/{envelope_id}")
    if response.json()["envelope"]["status"] == "SUCCESS":
        break
    time.sleep(30)

# GOOD: Webhooks
# Set up webhook to receive envelope.signed event
# Your endpoint receives notification when envelope completes
```

See [Webhook Integration Guide](../guides/webhook-integration.md).

### 2. Validate Files Before Upload

Ensure PDFs are valid before creating envelopes:

```python
import PyPDF2

def validate_pdf(file_path):
    try:
        with open(file_path, 'rb') as f:
            PyPDF2.PdfReader(f)
        return True
    except:
        return False

# Validate before encoding
if validate_pdf('contract.pdf'):
    # Create envelope
    pass
else:
    print("Invalid PDF file")
```

### 3. Handle Errors Gracefully

Implement proper error handling:

```python
try:
    response = requests.post(f"{API_BASE}/envelopes", headers=headers, json=data)
    response.raise_for_status()
    envelope = response.json()["envelope"]
except requests.HTTPError as e:
    if e.response.status_code == 422:
        # Validation error
        errors = e.response.json()
        print(f"Validation failed: {errors}")
    elif e.response.status_code == 401:
        # Invalid API key
        print("Authentication failed")
    else:
        print(f"Error: {e}")
```

### 4. Store Envelope IDs

Always save envelope IDs for later reference:

```python
# Save to database
db.envelopes.insert({
    'envelope_id': envelope_id,
    'created_at': datetime.now(),
    'status': 'CREATED',
    'recipient_email': 'john.doe@example.com'
})
```

---

## Troubleshooting

### Error: "Envelope not in CREATED status"

**Cause:** Trying to send an envelope that's already been sent.

**Solution:** Check envelope status before sending:

```python
status_response = requests.get(f"{API_BASE}/envelopes/{envelope_id}")
status = status_response.json()["envelope"]["status"]

if status == "CREATED":
    # Safe to send
    send_response = requests.post(f"{API_BASE}/envelopes/{envelope_id}/send_invitation")
```

### Error: "Invalid PDF content"

**Cause:** Base64 encoding is incorrect or file is not a valid PDF.

**Solution:** Ensure proper encoding:

```python
# Correct encoding
with open('file.pdf', 'rb') as f:  # Note: 'rb' for binary mode
    content = base64.b64encode(f.read()).decode('utf-8')

# Verify it's valid base64
try:
    base64.b64decode(content)
except:
    print("Invalid base64 encoding")
```

### Slow Evidence Generation

**Cause:** First evidence request generates PDF synchronously.

**Solution:** This is expected. Subsequent requests will be fast (<100ms).

---

## Related Documentation

- [Quick Start Guide](../getting-started/quick-start.md) - Create your first envelope
- [Recipients API](recipients.md) - Manage envelope signers
- [Evidence Sheet Guide](../guides/evidence-sheet.md) - Download audit trails
- [Webhook Integration](../guides/webhook-integration.md) - Real-time notifications
- [Email Validation (2FA)](validations.md) - Add email verification

---

## Rate Limits

Envelope creation is subject to rate limits based on your plan:

| Plan | Envelopes/Hour | Envelopes/Day |
|------|----------------|---------------|
| Starter | 10 | 100 |
| Professional | 50 | 500 |
| Enterprise | Custom | Custom |

Contact support to increase limits.
