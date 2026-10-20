# Envelopes API

Envelopes are the core resource in LexgoSign. An envelope contains documents and recipients (signers), and manages the complete signature workflow from creation to completion.

## Overview

An envelope represents a signature request with one or more documents and recipients. Envelopes progress through several states:

```
CREATED → SENDING → IN_PROGRESS → SUCCESS
             ↓           ↓
           ERROR       VOIDED
```

### Envelope States

| State | Description |
|-------|-------------|
| `CREATED` | Envelope created but not sent to recipients |
| `SENDING` | Send accepted; asynchronous processing underway |
| `ERROR` | Validation or processing failure; inspect errors |
| `IN_PROGRESS` | Envelope sent, awaiting signatures |
| `SUCCESS` | All recipients have signed |
| `VOIDED` | Envelope cancelled before completion |

## Endpoints

### Create Envelope

Create a new envelope with documents and recipients. Valid creation returns HTTP
200. Synchronous validation failures return HTTP 422 with the same envelope
response, including its ID, `status: ERROR`, `errors`, and `request_id`.
Keep that ID and correct the envelope with `PUT /v1/envelopes/:id` before sending.

**Endpoint:** `POST /v1/envelopes`

=== "Request"
    ```http
    POST /v1/envelopes HTTP/1.1
    Host: api.lexgo.cl
    Authorization: YOUR_API_KEY
    Content-Type: multipart/form-data

    name=Employment Contract - John Doe
    documents[0][base64]=JVBERi0xLjQKJeLj...
    documents[0][name]=employment-contract.pdf
    recipients[0][name]=John Doe
    recipients[0][email]=john.doe@example.com
    recipients[0][order]=1
    placements[0][document_key]=0
    placements[0][recipient_key]=0
    placements[0][type]=SIGNATURE
    ```

=== "Response (200 OK)"
    ```json
    {
      "envelope": {
        "id": "abc-123-def-456",
        "name": "Employment Contract - John Doe",
        "status": "CREATED",
        "created_at": "2024-01-15T10:30:00Z",
        "recipients": [
          {
            "id": "signer-001",
            "name": "John Doe",
            "email": "john.doe@example.com",
            "order": 1,
            "status": "CREATED"
          }
        ],
        "metadata": {}
      },
      "request_id": "req-abc123"
    }
    ```

=== "Response (422 Unprocessable Content)"
    ```json
    {
      "envelope": {
        "id": "abc-123-def-456",
        "name": "Employment Contract - John Doe",
        "status": "ERROR",
        "errors": ["There are no signature placements that relate recipients to their respective documents."],
        "warnings": []
      },
      "request_id": "req-abc123"
    }
    ```

A 422 response can still contain a persisted envelope. Correct that envelope using
its ID rather than retrying `POST`, which creates another envelope. Fetch/update
can still return HTTP 200 with `status: ERROR`; inspect the validation fields.

**Request Parameters:**

!!! note "Parameter Format"
    This API accepts JSON or **multipart/form-data** with bracket notation for nested parameters. You can use numeric indices like `recipients[0]` or custom keys like `recipients[john]`.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | No | Envelope name/title (defaults to first document name) |
| `documents[key][...]` | object | Yes | Document parameters (see below) |
| `recipients[key][...]` | object | Yes | Recipient parameters (see below) |
| `placements[index][...]` | object | Yes | Placement parameters (see below) |
| `settings[...]` | object | No | Override default settings (see [Settings API](settings.md)) |
| `metadata` | object | No | Your own JSON object describing this envelope (customer, reference, cost center...). See [Envelope metadata](#envelope-metadata) |

### Envelope metadata

`metadata` is an optional JSON object that **you** define when you create an envelope.
Lexgo stores it as opaque context. It never changes recipients, signing, permissions,
pricing or billing. Its purpose is to let you attribute usage to your own end customers:
the same object appears on the matching row of the FES and FEA Excel usage reports (see
[Usage reports](../guides/usage-reports.md)). You keep the mapping between your keys and
your end customers; Lexgo does not interpret them.

**Accepted values.** `metadata` must be a JSON object. Its values can be any valid JSON:
strings, numbers, booleans, `null`, arrays and nested objects. Unicode is preserved. Strings and keys cannot contain the NUL character (`\u0000`), which the storage cannot hold.

**Numbers.** Integers are kept exactly. A number written with a fraction or an exponent is read as a 64-bit float (about 15 to 17 significant digits), so send exact decimals as strings. A nonzero number too large or too small for a 64-bit float, such as `1e400` or `1e-400`, is rejected with HTTP 422; no number is ever changed to zero, infinity or a string. The database stores numbers as decimals and returns them in expanded notation: `1e20` comes back as `100000000000000000000` and `1.5e300` as a 301-digit integer. The 8,192-byte limit is therefore checked against the object as it is returned after storage, so metadata with many large exponents is rejected even when the request itself is short.
Send it in a JSON body to keep value types. In `multipart/form-data` use bracket
notation (`metadata[customer_id]=customer-123`); form values are always strings.
Key order is not preserved: Lexgo returns the same keys and values, ordered by its own
storage rules.

**Limits.** Metadata is validated before the envelope is created. Nothing is truncated.

| Limit | Value |
|-------|-------|
| Serialized size | 8,192 bytes (UTF-8 JSON, counted in bytes, not characters) |
| Nesting depth | 5 levels (the root object is level 1; every nested object or array adds one) |
| Keys | 100 in total across all levels |

**Omitted or invalid.** If you omit `metadata`, the envelope is created normally and
`metadata` reads back as `{}`. Envelopes created before this field existed also read as
`{}`. A value that is not an object (a string, array, number or `null`) or that exceeds a
limit is rejected with HTTP 422 and **no envelope is created**. Unlike an envelope
validation failure, this response has no envelope ID to correct: fix the request and send
it again.

```json title="metadata-error-response"
{
  "error": "Invalid metadata",
  "message": "metadata is nested 6 levels deep; the maximum is 5",
  "request_id": "req-abc123"
}
```

**Scope.** Metadata belongs to the API key that created the envelope. It is returned only
by authenticated requests made with that key (`POST`, `GET`, `PUT` and send/void
responses). It is not shown to signers, not included in webhook payloads, not searchable,
and cannot be changed after creation. API request logs keep the request body you sent,
so do not put secrets in metadata.

**Complete example.** Create an envelope for one of your end customers:

```json title="metadata-create-request"
{
  "name": "Service agreement - Acme Ltda.",
  "documents": {
    "0": { "name": "service-agreement.pdf", "base64": "BASE64_PDF_CONTENT" }
  },
  "recipients": {
    "0": { "name": "Ana Rojas", "email": "ana.rojas@example.com", "order": 1 }
  },
  "placements": {
    "0": {
      "document_key": "0",
      "recipient_key": "0",
      "type": "SIGNATURE",
      "coordinates": { "page": 0, "top": 650, "left": 100 }
    }
  },
  "metadata": {
    "customer_id": "customer-123",
    "cost_center": "sales",
    "reference": "invoice-456",
    "contract": { "id": "C-2026-0042", "tags": ["renewal", "premium"] }
  }
}
```

The response echoes the stored object:

```json title="metadata-create-response"
{
  "envelope": {
    "id": "abc-123-def-456",
    "name": "Service agreement - Acme Ltda.",
    "status": "CREATED",
    "metadata": {
      "customer_id": "customer-123",
      "cost_center": "sales",
      "reference": "invoice-456",
      "contract": { "id": "C-2026-0042", "tags": ["renewal", "premium"] }
    }
  },
  "request_id": "req-abc123"
}
```

Retrieve it later with the same API key:

```bash
curl https://api.lexgo.cl/v1/envelopes/abc-123-def-456 \
  -H "Authorization: YOUR_API_KEY"
```

```json title="metadata-get-response"
{
  "envelope": {
    "id": "abc-123-def-456",
    "status": "CREATED",
    "metadata": {
      "customer_id": "customer-123",
      "cost_center": "sales",
      "reference": "invoice-456",
      "contract": { "id": "C-2026-0042", "tags": ["renewal", "premium"] }
    }
  },
  "request_id": "req-def456"
}
```

**In the Excel usage reports.** The FES and FEA reports have a detail sheet, *Detalle de
firmas*, with one row per signed envelope. Both reports end that sheet with a single
**Metadata** column holding the envelope's metadata as one line of JSON text, for example
`{"cost_center":"sales","customer_id":"customer-123", ...}`. It is the same column in the
reports Lexgo generates every month for billing clients and in any report generated on
request, because both use the same generator. Parse the cell with any JSON parser and look
up your own keys (for example `JSON.parse(cell)["customer_id"]`). The cell is plain text,
never a formula, and keeps Unicode and nested values. Envelopes without metadata, including
all envelopes created before this field existed, have an empty cell. The summary and
pricing sheets are unchanged: they describe many envelopes, so they carry no metadata, and
totals are not grouped by metadata. Lexgo does not offer a column per metadata key,
searching or filtering by metadata, retroactive metadata for older envelopes, or metadata
in webhooks.

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

### Placement Parameters (Required)

Placements associate each recipient with a document and action. At least one placement is required, every recipient must have one, and each original document must have one. Omitting the entire collection is a validation error. A SIGNATURE placement without coordinates/token uses an extra signature page; it does not remove the placement requirement.

Use `placements[index][field]` where `index` is a numeric index (0, 1, 2...).

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `placements[index][document_key]` | string | Yes | Document key from `documents[key]` |
| `placements[index][recipient_key]` | string | Yes | Recipient key from `recipients[key]` |
| `placements[index][type]` | string | No | Field type: `SIGNATURE` (default), `APPROVER`, `WITNESS` |
| `placements[index][order]` | integer | No | Display order (defaults to index) |
| `placements[index][coordinates][top]` | decimal | No* | Distance from top of PDF page (PDF coordinate units) |
| `placements[index][coordinates][left]` | decimal | No* | Distance from left of PDF page (PDF coordinate units) |
| `placements[index][coordinates][page]` | integer | No* | Page number (0-indexed, 0 = first page) |
| `placements[index][token]` | string | No* | Token to position signature (alternative to coordinates) |

!!! note "Positioning"
    For SIGNATURE placements, coordinates (top/left/page) or a token position the signature. If neither is provided, signatures are added on extra pages. APPROVER and WITNESS placements must omit both coordinates and token. ATTACHMENT documents cannot have SIGNATURE placements. Do not mix placement types for the same document/recipient pair; at most one APPROVER or WITNESS is allowed for that pair.

**Example with Placements:**

```http
POST /v1/envelopes HTTP/1.1
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

**Endpoint:** `GET /v1/envelopes/:id`

=== "Request"
    ```bash
    curl https://api.lexgo.cl/v1/envelopes/abc-123-def-456 \
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
        "recipients": [
          {
            "id": "signer-001",
            "name": "John Doe",
            "email": "john.doe@example.com",
            "order": 1,
            "status": "SIGNED_ALL",
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
| `status` | string | Current lifecycle status, including CREATED, SENDING, IN_PROGRESS, SUCCESS, VOIDED, ERROR |
| `created_at` | datetime | When envelope was created |
| `sent_at` | datetime | When envelope was sent to recipients |
| `completed_at` | datetime | When all signatures completed |
| `recipients` | array | List of recipients with their status |
| `metadata` | object | The caller-defined object sent at creation; `{}` when none was sent (see [Envelope metadata](#envelope-metadata)) |

---

### Update Envelope

Update an existing envelope before it's sent to recipients.

**Endpoint:** `PUT /v1/envelopes/:id`

!!! info "Update Restrictions"
    Envelopes can only be updated while in `CREATED` or `ERROR` status. Once sent (`IN_PROGRESS`), updates are no longer allowed.

!!! tip "Error Recovery"
    If an envelope is in `ERROR` status and you fix the validation errors, the status will automatically transition back to `CREATED`.

=== "Request"
    ```http
    PUT /v1/envelopes/abc-123 HTTP/1.1
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

Queue asynchronous envelope processing.

**Endpoint:** `PUT /v1/envelopes/:id/send`

The envelope must be `CREATED` and have no validation errors. No body is
required. Sending returns immediately; it does not prove delivery or completion.

```bash
curl -X PUT https://api.lexgo.cl/v1/envelopes/ENVELOPE_ID/send \
  -H "Authorization: YOUR_API_KEY"
```

**HTTP 202 Accepted** (abridged):

```json
{
  "success": true,
  "envelope": {"id": "envelope-id", "status": "SENDING", "errors": [], "warnings": []},
  "message": "Envelope is being processed. Use webhooks or poll for status updates.",
  "request_id": "request-id"
}
```

Processing prepares documents and invitations asynchronously. If invitation
email is disabled, signing links are generated without mailing recipients.
Observe subsequent state through webhooks and reconcile with GET. A 405 with
`error: "Envelope not in CREATED status"` can mean the envelope was already
sent **or** has validation errors; retrieve it and inspect `errors`.
A missing envelope returns 404. Do not blindly retry after a network timeout.

---

### Void Envelope

Cancel an envelope before completion.

**Endpoint:** `PUT /v1/envelopes/:id/void`

!!! warning "Irreversible"
    Voiding an envelope cannot be undone. Recipients will no longer be able to access or sign the documents.

=== "Request"
    ```bash
    curl -X PUT https://api.lexgo.cl/v1/envelopes/abc-123/void \
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

**Endpoint:** `GET /v1/envelopes/:id/evidence`

!!! success "Optimized Performance"
    This endpoint uses intelligent async caching for **20-50x faster** response times on subsequent requests.

=== "Request"
    ```bash
    curl https://api.lexgo.cl/v1/envelopes/abc-123/evidence \
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

Check HTTP status and `envelope.errors` after create/update. Create validation failures return HTTP 422; updates can still return HTTP 200 with an invalid envelope. After send, track `SENDING` through processing; `success: true` means accepted, not signed.

### Complete Workflow

=== "Python"
    ```python
    import requests
    import base64
    import time

    API_BASE = "https://api.lexgo.cl/v1"
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
        "recipients[0][order]": "1",
        "placements[0][document_key]": "0",
        "placements[0][recipient_key]": "0",
        "placements[0][type]": "SIGNATURE"
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
    send_response = requests.put(
        f"{API_BASE}/envelopes/{envelope_id}/send",
        headers=headers
    )

    if send_response.json()["success"]:
        print("✓ Envelope send accepted; reconcile status")

    # 3. Check status
    status_response = requests.get(
        f"{API_BASE}/envelopes/{envelope_id}",
        headers=headers
    )

    status = status_response.json()["envelope"]["status"]
    print(f"✓ Current status: {status}")

    # 4. Wait for completion and get evidence (optional)
    # Combine webhooks with periodic polling/reconciliation
    while status not in ["SUCCESS", "VOIDED", "ERROR"]:
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

    const API_BASE = 'https://api.lexgo.cl/v1';
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
      formData.append('placements[0][document_key]', '0');
      formData.append('placements[0][recipient_key]', '0');
      formData.append('placements[0][type]', 'SIGNATURE');

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
        `${API_BASE}/envelopes/${envelopeId}/send`,
        {
          method: 'PUT',
          headers: {
            'Authorization': API_KEY
          }
        }
      );

      const { success } = await sendResponse.json();
      if (success) {
        console.log('✓ Envelope send accepted; reconcile status');
      }

      return envelopeId;
    }

    createAndSendEnvelope();
    ```

=== "Ruby"
    ```ruby
    require 'net/http'
    require 'base64'

    API_BASE = 'https://api.lexgo.cl/v1'
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
      ['recipients[0][order]', '1'],
      ['placements[0][document_key]', '0'],
      ['placements[0][recipient_key]', '0'],
      ['placements[0][type]', 'SIGNATURE']
    ]

    uri = URI("#{API_BASE}/envelopes")
    request = Net::HTTP::Put.new(uri)
    request['Authorization'] = API_KEY
    request.set_form(form_data, 'multipart/form-data')

    response = Net::HTTP.start(uri.hostname, uri.port, use_ssl: true) do |http|
      http.request(request)
    end

    envelope = JSON.parse(response.body)['envelope']
    envelope_id = envelope['id']
    puts "✓ Created envelope: #{envelope_id}"

    # 2. Send to recipients
    uri = URI("#{API_BASE}/envelopes/#{envelope_id}/send")
    request = Net::HTTP::Put.new(uri)
    request['Authorization'] = API_KEY

    response = Net::HTTP.start(uri.hostname, uri.port, use_ssl: true) do |http|
      http.request(request)
    end

    result = JSON.parse(response.body)
    puts '✓ Envelope send accepted; reconcile status' if result['success']
    ```

---

## Best Practices

### 1. Combine Webhooks with Reconciliation

Use [webhooks](../guides/webhook-integration.md) for prompt updates and
periodically retrieve stored envelope IDs to recover from missed or delayed
notifications. Webhooks have bounded best-effort retries. Deduplicate delivery
IDs and make downstream business operations idempotent.

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
    if envelope.get("errors") or envelope["status"] == "ERROR":
        print(f"Envelope validation failed: {envelope.get('errors')}")
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

**Cause:** The envelope was already sent or has validation errors. Retrieve it and inspect status/errors before retrying.

**Solution:** Check envelope status before sending:

```python
status_response = requests.get(f"{API_BASE}/envelopes/{envelope_id}")
status = status_response.json()["envelope"]["status"]

if status == "CREATED" and not status_response.json()["envelope"].get("errors"):
    # Safe to send
    send_response = requests.put(f"{API_BASE}/envelopes/{envelope_id}/send")
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
