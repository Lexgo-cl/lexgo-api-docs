# Evidence Sheet Guide

Evidence sheets provide complete audit trails for legal compliance, including a comprehensive timeline of all signature events, signer authentication details, and cryptographic verification for legal validity.

!!! success "Optimized Performance"
    The evidence endpoint uses intelligent async caching for **20-50x faster** response times on subsequent requests.

## Overview

Evidence sheets are legally binding PDF documents that include:

- **Complete timeline** of all envelope events with timestamps
- **IP addresses** and geolocation data for each action
- **Email delivery confirmations** and read receipts
- **Signature verification** with cryptographic details
- **Authentication records** including 2FA validation attempts
- **Cryptographic hashes** for document integrity
- **Legal compliance** information and audit trail

## How Evidence Generation Works

### Async Caching Architecture

The evidence endpoint uses a smart caching pattern for optimal performance:

| Scenario | Response Time | Behavior |
|----------|--------------|----------|
| **First Request** | 1-3 seconds | Generates evidence synchronously |
| **Subsequent Requests** | <100ms | Returns cached URL immediately (20-50x faster!) |
| **Background Updates** | Non-blocking | Auto-regenerates when timeline changes |

### Automatic Generation

Evidence sheets are automatically generated when:

1. ✅ Envelope status changes to `SUCCESS`
2. ✅ Timeline has new events since last generation
3. ✅ Manual regeneration is triggered

Evidence is NOT regenerated when timeline is unchanged, avoiding unnecessary processing.

## Getting Evidence Sheets

### Basic Request

Once an envelope reaches `SUCCESS` status:

=== "cURL"
    ```bash
    curl https://api.lexgo.cl/api/v1/envelopes/{envelope_id}/evidence \
      -H "Authorization: Bearer YOUR_API_KEY"
    ```

=== "Python"
    ```python
    import requests

    envelope_id = "abc-123"
    headers = {"Authorization": f"Bearer {API_KEY}"}

    response = requests.get(
        f"https://api.lexgo.cl/api/v1/envelopes/{envelope_id}/evidence",
        headers=headers
    )

    data = response.json()
    evidence_url = data["evidence_document_url"]
    ```

=== "JavaScript"
    ```javascript
    const envelopeId = 'abc-123';
    const response = await fetch(
      `https://api.lexgo.cl/api/v1/envelopes/${envelopeId}/evidence`,
      {
        headers: {
          'Authorization': `Bearer ${API_KEY}`
        }
      }
    );

    const data = await response.json();
    const evidenceUrl = data.evidence_document_url;
    ```

### Success Response

```json
{
  "success": true,
  "evidence_document_url": "https://s3.amazonaws.com/.../evidence_sheet.pdf",
  "request_id": "req-001"
}
```

The `evidence_document_url` is a direct download link to the PDF file stored in S3.

### Error Responses

```json
// Envelope not completed yet
{
  "error": "Envelope not in SUCCESS status",
  "request_id": "req-002"
}

// Envelope not found or unauthorized
{
  "error": "Envelope not found",
  "request_id": "req-003"
}

// Evidence generation failed
{
  "success": false,
  "evidence_document_url": null,
  "request_id": "req-004"
}
```

## Downloading the PDF

### Option 1: Direct Download

The `evidence_document_url` can be used directly in a browser or download tool:

```bash
# Download with curl
curl -o evidence.pdf "https://s3.amazonaws.com/.../evidence_sheet.pdf"

# Download with wget
wget -O evidence.pdf "https://s3.amazonaws.com/.../evidence_sheet.pdf"
```

### Option 2: Programmatic Download

=== "Python"
    ```python
    import requests

    # Get evidence URL
    response = requests.get(f"{API_BASE}/envelopes/{envelope_id}/evidence", headers=headers)
    evidence_url = response.json()["evidence_document_url"]

    # Download PDF
    pdf_response = requests.get(evidence_url)
    with open(f"evidence_{envelope_id}.pdf", "wb") as f:
        f.write(pdf_response.content)

    print(f"Evidence saved to evidence_{envelope_id}.pdf")
    ```

=== "JavaScript"
    ```javascript
    // Get evidence URL
    const response = await fetch(`${API_BASE}/envelopes/${envelopeId}/evidence`, {
      headers: { 'Authorization': `Bearer ${API_KEY}` }
    });
    const { evidence_document_url } = await response.json();

    // Download PDF
    const pdfResponse = await fetch(evidence_document_url);
    const blob = await pdfResponse.blob();

    // Save to file (Node.js)
    const fs = require('fs');
    const buffer = await blob.arrayBuffer();
    fs.writeFileSync(`evidence_${envelopeId}.pdf`, Buffer.from(buffer));
    ```

=== "Ruby"
    ```ruby
    require 'net/http'
    require 'json'

    # Get evidence URL
    uri = URI("#{API_BASE}/envelopes/#{envelope_id}/evidence")
    request = Net::HTTP::Get.new(uri)
    request['Authorization'] = "Bearer #{API_KEY}"
    response = Net::HTTP.start(uri.hostname, uri.port, use_ssl: true) do |http|
      http.request(request)
    end

    evidence_url = JSON.parse(response.body)['evidence_document_url']

    # Download PDF
    pdf_content = Net::HTTP.get(URI(evidence_url))
    File.write("evidence_#{envelope_id}.pdf", pdf_content)

    puts "Evidence saved to evidence_#{envelope_id}.pdf"
    ```

## Best Practices

### 1. Check Envelope Status First

Always verify the envelope is complete before requesting evidence:

```python
# Get envelope status
response = requests.get(f"{API_BASE}/envelopes/{envelope_id}", headers=headers)
envelope = response.json()["envelope"]

# Only request evidence if SUCCESS
if envelope["status"] == "SUCCESS":
    evidence_response = requests.get(
        f"{API_BASE}/envelopes/{envelope_id}/evidence",
        headers=headers
    )
else:
    print(f"Envelope not ready: {envelope['status']}")
```

### 2. Cache Evidence URLs

Evidence URLs are stable and can be cached locally:

```python
# Store URL for later use
evidence_cache[envelope_id] = evidence_url

# No need to poll the endpoint repeatedly
# Just download from cached URL when needed
```

### 3. Handle Errors Gracefully

Implement proper error handling:

```python
try:
    response = requests.get(f"{API_BASE}/envelopes/{envelope_id}/evidence", headers=headers)
    response.raise_for_status()

    data = response.json()
    if data.get("success"):
        return data["evidence_document_url"]
    else:
        # Generation failed
        logger.error(f"Evidence generation failed for {envelope_id}")
        return None

except requests.HTTPError as e:
    if e.response.status_code == 405:
        # Envelope not ready yet
        logger.info(f"Envelope {envelope_id} not completed")
    elif e.response.status_code == 404:
        # Envelope not found
        logger.error(f"Envelope {envelope_id} not found")
    return None
```

### 4. Download PDF Immediately

Don't rely on long-term URL storage:

```python
# GOOD: Download and store PDF
evidence_url = get_evidence_url(envelope_id)
pdf_content = download_pdf(evidence_url)
store_in_database(envelope_id, pdf_content)

# BAD: Only store URL long-term
evidence_url = get_evidence_url(envelope_id)
store_in_database(envelope_id, evidence_url)  # URL might change!
```

## Legal Compliance

Evidence sheets are legally binding and compliant with:

- **Chilean Law 19.799** - Electronic Documents and Digital Signatures
- **ESIGN Act** - United States Electronic Signatures in Global and National Commerce Act
- **eIDAS Regulation** - European Union Electronic Identification and Trust Services

Evidence sheets include:

- Complete audit trail with timestamps
- Cryptographic verification details
- Signer authentication records
- Document integrity validation
- IP addresses and geolocation data

## Troubleshooting

### "Envelope not in SUCCESS status" Error

**Cause**: Envelope is still in progress or failed.

**Solution**: Check envelope status and wait for completion:

```python
envelope = get_envelope(envelope_id)
print(f"Status: {envelope['status']}")
# Wait until status is SUCCESS
```

### Slow Response Times

**Cause**: First request generates evidence synchronously.

**Solution**: This is expected behavior. Subsequent requests will be fast (<100ms).

### Evidence Missing Recent Events

**Cause**: Evidence was cached before latest events.

**Solution**: Wait 30-60 seconds and request again. Background update will have completed.

### Generation Failed (500 Error)

**Cause**: PDF generation or S3 upload failed.

**Solution**:
- Retry the request after a few seconds
- Check envelope has valid timeline data
- Contact support if issue persists

## Related Documentation

- [Envelopes API Reference](../api/envelopes.md) - Complete envelope endpoint documentation
- [Webhooks Guide](webhook-integration.md) - Get notified when envelope completes
- [API Overview](../api/overview.md) - General API conventions

## Performance Tips

### For High-Volume Applications

1. **Use Webhooks**: Get notified when envelope completes instead of polling
2. **Cache Locally**: Store evidence URLs to avoid repeated API calls
3. **Batch Downloads**: Download multiple evidence sheets in parallel
4. **Monitor Workers**: Track evidence generation metrics

### Expected Performance

- First evidence request: 1-3 seconds (generation time)
- Cached requests: <100ms (20-50x faster)
- Background updates: Non-blocking, happens asynchronously

## Example: Complete Workflow

```python
import requests
import time

API_BASE = "https://api.lexgo.cl/api/v1"
headers = {"Authorization": f"Bearer {API_KEY}"}

def wait_for_envelope_completion(envelope_id, timeout=3600):
    """Wait for envelope to reach SUCCESS status."""
    start_time = time.time()

    while time.time() - start_time < timeout:
        response = requests.get(f"{API_BASE}/envelopes/{envelope_id}", headers=headers)
        envelope = response.json()["envelope"]

        if envelope["status"] == "SUCCESS":
            return True
        elif envelope["status"] in ["VOIDED", "EXPIRED"]:
            return False

        time.sleep(30)  # Poll every 30 seconds

    return False

def download_evidence(envelope_id):
    """Download evidence sheet for completed envelope."""
    # Get evidence URL
    response = requests.get(
        f"{API_BASE}/envelopes/{envelope_id}/evidence",
        headers=headers
    )

    if response.status_code == 200:
        data = response.json()
        if data["success"]:
            evidence_url = data["evidence_document_url"]

            # Download PDF
            pdf_response = requests.get(evidence_url)
            filename = f"evidence_{envelope_id}.pdf"

            with open(filename, "wb") as f:
                f.write(pdf_response.content)

            print(f"✓ Evidence downloaded: {filename}")
            return filename

    return None

# Complete workflow
envelope_id = "abc-123"

print(f"Waiting for envelope {envelope_id} to complete...")
if wait_for_envelope_completion(envelope_id):
    print("✓ Envelope completed successfully")
    evidence_file = download_evidence(envelope_id)
    print(f"✓ Evidence saved to: {evidence_file}")
else:
    print("✗ Envelope did not complete")
```
