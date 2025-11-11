# Quick Start

This guide will walk you through creating and sending your first envelope with LexgoSign in just a few minutes.

## Prerequisites

Before you begin, make sure you have:

- A Lexgo account
- An API key ([get one here](authentication.md))
- A PDF document to send for signature
- Recipient email address

## Step 1: Create an Envelope

Send a POST request to create an envelope with your document and recipients.

=== "cURL"
    ```bash
    curl -X POST https://api.lexgo.cl/api/v1/envelopes \
      -H "Authorization: YOUR_API_KEY" \
      -F "name=Employment Contract - John Doe" \
      -F "documents[0][base64]=JVBERi0xLjUKJc/s/+jXy80KMSAwIG9iago8PC9UeXBlL1BhZ2UvTWVkaWFCb3hbMCAwIDU5NS4yNzU1OTEgODQxLjg4OTc2NF0vUGFyZW50IDMgMCBSL1Jlc291cmNlczw8L0ZvbnQ8PC9GMSA2IDAgUj4+Pj4vQ29udGVudHMgNCAwIFI+PgplbmRvYmoKMiAwIG9iago8PC9UeXBlL0NhdGFsb2cvUGFnZXMgMyAwIFI+PgplbmRvYmoKMyAwIG9iago8PC9UeXBlL1BhZ2VzL0tpZHNbMSAwIFJdL0NvdW50IDE+PgplbmRvYmoKNCAwIG9iago8PC9GaWx0ZXIvRmxhdGVEZWNvZGUvTGVuZ3RoIDYzPj5zdHJlYW0KeNrTdzNUMDJQCEnjcgrhMlQwAEJDBRMDBXMLoGAuV7RGQFFpalKigk9qRXq+QnBmep5mbIgXl2sIFwCBuw6ACmVuZHN0cmVhbQplbmRvYmoKNSAwIG9iago8PC9UeXBlL0ZvbnREZXNjcmlwdG9yL0ZvbnROYW1lL0hlbHZldGljYS9Gb250V2VpZ2h0IDUwMC9Gb250QkJveFstMTY2LjAgLTIyNS4wIDEwMDAuMCA5MzEuMF0vSXRhbGljQW5nbGUgMC4wL0FzY2VudCA3MTguMC9EZXNjZW50IC0yMDcuMC9DYXBIZWlnaHQgNzE4LjAvWEhlaWdodCA1MjMuMC9TdGVtSCA3Ni4wL1N0ZW1WIDg4LjAvRmxhZ3MgMzI+PgplbmRvYmoKNiAwIG9iago8PC9UeXBlL0ZvbnQvU3VidHlwZS9UeXBlMS9CYXNlRm9udC9IZWx2ZXRpY2EvRm9udERlc2NyaXB0b3IgNSAwIFIvRmlyc3RDaGFyIDMyL0xhc3RDaGFyIDI1NS9XaWR0aHNbMjc4LjAgMjc4LjAgMzU1LjAgNTU2LjAgNTU2LjAgODg5LjAgNjY3LjAgMTkxLjAgMzMzLjAgMzMzLjAgMzg5LjAgNTg0LjAgMjc4LjAgMzMzLjAgMjc4LjAgMjc4LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgMjc4LjAgMjc4LjAgNTg0LjAgNTg0LjAgNTg0LjAgNTU2LjAgMTAxNS4wIDY2Ny4wIDY2Ny4wIDcyMi4wIDcyMi4wIDY2Ny4wIDYxMS4wIDc3OC4wIDcyMi4wIDI3OC4wIDUwMC4wIDY2Ny4wIDU1Ni4wIDgzMy4wIDcyMi4wIDc3OC4wIDY2Ny4wIDc3OC4wIDcyMi4wIDY2Ny4wIDYxMS4wIDcyMi4wIDY2Ny4wIDk0NC4wIDY2Ny4wIDY2Ny4wIDYxMS4wIDI3OC4wIDI3OC4wIDI3OC4wIDQ2OS4wIDU1Ni4wIDMzMy4wIDU1Ni4wIDU1Ni4wIDUwMC4wIDU1Ni4wIDU1Ni4wIDI3OC4wIDU1Ni4wIDU1Ni4wIDIyMi4wIDIyMi4wIDUwMC4wIDIyMi4wIDgzMy4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDMzMy4wIDUwMC4wIDI3OC4wIDU1Ni4wIDUwMC4wIDcyMi4wIDUwMC4wIDUwMC4wIDUwMC4wIDMzNC4wIDI2MC4wIDMzNC4wIDU4NC4wIDM1MC4wIDU1Ni4wIDM1MC4wIDIyMi4wIDU1Ni4wIDMzMy4wIDEwMDAuMCA1NTYuMCA1NTYuMCAzMzMuMCAxMDAwLjAgNjY3LjAgMzMzLjAgMTAwMC4wIDM1MC4wIDYxMS4wIDM1MC4wIDM1MC4wIDIyMi4wIDIyMi4wIDMzMy4wIDMzMy4wIDM1MC4wIDU1Ni4wIDEwMDAuMCAzMzMuMCAxMDAwLjAgNTAwLjAgMzMzLjAgOTQ0LjAgMzUwLjAgNTAwLjAgNjY3LjAgMjc4LjAgMzMzLjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgMjYwLjAgNTU2LjAgMzMzLjAgNzM3LjAgMzcwLjAgNTU2LjAgNTg0LjAgMzMzLjAgNzM3LjAgMzMzLjAgNDAwLjAgNTg0LjAgMzMzLjAgMzMzLjAgMzMzLjAgNTU2LjAgNTM3LjAgMjc4LjAgMzMzLjAgMzMzLjAgMzY1LjAgNTU2LjAgODM0LjAgODM0LjAgODM0LjAgNjExLjAgNjY3LjAgNjY3LjAgNjY3LjAgNjY3LjAgNjY3LjAgNjY3LjAgMTAwMC4wIDcyMi4wIDY2Ny4wIDY2Ny4wIDY2Ny4wIDY2Ny4wIDI3OC4wIDI3OC4wIDI3OC4wIDI3OC4wIDcyMi4wIDcyMi4wIDc3OC4wIDc3OC4wIDc3OC4wIDc3OC4wIDc3OC4wIDU4NC4wIDc3OC4wIDcyMi4wIDcyMi4wIDcyMi4wIDcyMi4wIDY2Ny4wIDY2Ny4wIDYxMS4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDg4OS4wIDUwMC4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDI3OC4wIDI3OC4wIDI3OC4wIDI3OC4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU4NC4wIDYxMS4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDUwMC4wIDU1Ni4wIDUwMC4wXS9FbmNvZGluZy9XaW5BbnNpRW5jb2Rpbmc+PgplbmRvYmoKNyAwIG9iago8PC9Nb2REYXRlKEQ6MjAyNDExMjgxNjUxMDEpL1Byb2R1Y2VyKEhleGFQREYgdmVyc2lvbiAwLjQ2LjApPj4KZW5kb2JqCnhyZWYKMCA4CjAwMDAwMDAwMDAgNjU1MzUgZiAKMDAwMDAwMDAxOCAwMDAwMCBuIAowMDAwMDAwMTQ0IDAwMDAwIG4gCjAwMDAwMDAxODkgMDAwMDAgbiAKMDAwMDAwMDI0MCAwMDAwMCBuIAowMDAwMDAwMzY5IDAwMDAwIG4gCjAwMDAwMDA1ODUgMDAwMDAgbiAKMDAwMDAwMjA3OSAwMDAwMCBuIAp0cmFpbGVyCjw8L1Jvb3QgMiAwIFIvSURbKHtbPrtqnyTtl1Nt5AL9dhIpKHtbPrtqnyTtl1Ot5AL9dhIpXS9JbmZvIDcgMCBSL1NpemUgOD4+CnN0YXJ0eHJlZgoyMTU4CiUlRU9GCg==" \
      -F "documents[0][name]=employment-contract.pdf" \
      -F "recipients[0][name]=John Doe" \
      -F "recipients[0][email]=john.doe@example.com" \
      -F "recipients[0][order]=1"
    ```

=== "Python"
    ```python
    import requests

    # Test PDF (or read from file with: base64.b64encode(open('file.pdf', 'rb').read()).decode())
    test_pdf = "JVBERi0xLjUKJc/s/+jXy80KMSAwIG9iago8PC9UeXBlL1BhZ2UvTWVkaWFCb3hbMCAwIDU5NS4yNzU1OTEgODQxLjg4OTc2NF0vUGFyZW50IDMgMCBSL1Jlc291cmNlczw8L0ZvbnQ8PC9GMSA2IDAgUj4+Pj4vQ29udGVudHMgNCAwIFI+PgplbmRvYmoKMiAwIG9iago8PC9UeXBlL0NhdGFsb2cvUGFnZXMgMyAwIFI+PgplbmRvYmoKMyAwIG9iago8PC9UeXBlL1BhZ2VzL0tpZHNbMSAwIFJdL0NvdW50IDE+PgplbmRvYmoKNCAwIG9iago8PC9GaWx0ZXIvRmxhdGVEZWNvZGUvTGVuZ3RoIDYzPj5zdHJlYW0KeNrTdzNUMDJQCEnjcgrhMlQwAEJDBRMDBXMLoGAuV7RGQFFpalKigk9qRXq+QnBmep5mbIgXl2sIFwCBuw6ACmVuZHN0cmVhbQplbmRvYmoKNSAwIG9iago8PC9UeXBlL0ZvbnREZXNjcmlwdG9yL0ZvbnROYW1lL0hlbHZldGljYS9Gb250V2VpZ2h0IDUwMC9Gb250QkJveFstMTY2LjAgLTIyNS4wIDEwMDAuMCA5MzEuMF0vSXRhbGljQW5nbGUgMC4wL0FzY2VudCA3MTguMC9EZXNjZW50IC0yMDcuMC9DYXBIZWlnaHQgNzE4LjAvWEhlaWdodCA1MjMuMC9TdGVtSCA3Ni4wL1N0ZW1WIDg4LjAvRmxhZ3MgMzI+PgplbmRvYmoKNiAwIG9iago8PC9UeXBlL0ZvbnQvU3VidHlwZS9UeXBlMS9CYXNlRm9udC9IZWx2ZXRpY2EvRm9udERlc2NyaXB0b3IgNSAwIFIvRmlyc3RDaGFyIDMyL0xhc3RDaGFyIDI1NS9XaWR0aHNbMjc4LjAgMjc4LjAgMzU1LjAgNTU2LjAgNTU2LjAgODg5LjAgNjY3LjAgMTkxLjAgMzMzLjAgMzMzLjAgMzg5LjAgNTg0LjAgMjc4LjAgMzMzLjAgMjc4LjAgMjc4LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgMjc4LjAgMjc4LjAgNTg0LjAgNTg0LjAgNTg0LjAgNTU2LjAgMTAxNS4wIDY2Ny4wIDY2Ny4wIDcyMi4wIDcyMi4wIDY2Ny4wIDYxMS4wIDc3OC4wIDcyMi4wIDI3OC4wIDUwMC4wIDY2Ny4wIDU1Ni4wIDgzMy4wIDcyMi4wIDc3OC4wIDY2Ny4wIDc3OC4wIDcyMi4wIDY2Ny4wIDYxMS4wIDcyMi4wIDY2Ny4wIDk0NC4wIDY2Ny4wIDY2Ny4wIDYxMS4wIDI3OC4wIDI3OC4wIDI3OC4wIDQ2OS4wIDU1Ni4wIDMzMy4wIDU1Ni4wIDU1Ni4wIDUwMC4wIDU1Ni4wIDU1Ni4wIDI3OC4wIDU1Ni4wIDU1Ni4wIDIyMi4wIDIyMi4wIDUwMC4wIDIyMi4wIDgzMy4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDMzMy4wIDUwMC4wIDI3OC4wIDU1Ni4wIDUwMC4wIDcyMi4wIDUwMC4wIDUwMC4wIDUwMC4wIDMzNC4wIDI2MC4wIDMzNC4wIDU4NC4wIDM1MC4wIDU1Ni4wIDM1MC4wIDIyMi4wIDU1Ni4wIDMzMy4wIDEwMDAuMCA1NTYuMCA1NTYuMCAzMzMuMCAxMDAwLjAgNjY3LjAgMzMzLjAgMTAwMC4wIDM1MC4wIDYxMS4wIDM1MC4wIDM1MC4wIDIyMi4wIDIyMi4wIDMzMy4wIDMzMy4wIDM1MC4wIDU1Ni4wIDEwMDAuMCAzMzMuMCAxMDAwLjAgNTAwLjAgMzMzLjAgOTQ0LjAgMzUwLjAgNTAwLjAgNjY3LjAgMjc4LjAgMzMzLjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgMjYwLjAgNTU2LjAgMzMzLjAgNzM3LjAgMzcwLjAgNTU2LjAgNTg0LjAgMzMzLjAgNzM3LjAgMzMzLjAgNDAwLjAgNTg0LjAgMzMzLjAgMzMzLjAgMzMzLjAgNTU2LjAgNTM3LjAgMjc4LjAgMzMzLjAgMzMzLjAgMzY1LjAgNTU2LjAgODM0LjAgODM0LjAgODM0LjAgNjExLjAgNjY3LjAgNjY3LjAgNjY3LjAgNjY3LjAgNjY3LjAgNjY3LjAgMTAwMC4wIDcyMi4wIDY2Ny4wIDY2Ny4wIDY2Ny4wIDY2Ny4wIDI3OC4wIDI3OC4wIDI3OC4wIDI3OC4wIDcyMi4wIDcyMi4wIDc3OC4wIDc3OC4wIDc3OC4wIDc3OC4wIDc3OC4wIDU4NC4wIDc3OC4wIDcyMi4wIDcyMi4wIDcyMi4wIDcyMi4wIDY2Ny4wIDY2Ny4wIDYxMS4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDg4OS4wIDUwMC4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDI3OC4wIDI3OC4wIDI3OC4wIDI3OC4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU4NC4wIDYxMS4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDUwMC4wIDU1Ni4wIDUwMC4wXS9FbmNvZGluZy9XaW5BbnNpRW5jb2Rpbmc+PgplbmRvYmoKNyAwIG9iago8PC9Nb2REYXRlKEQ6MjAyNDExMjgxNjUxMDEpL1Byb2R1Y2VyKEhleGFQREYgdmVyc2lvbiAwLjQ2LjApPj4KZW5kb2JqCnhyZWYKMCA4CjAwMDAwMDAwMDAgNjU1MzUgZiAKMDAwMDAwMDAxOCAwMDAwMCBuIAowMDAwMDAwMTQ0IDAwMDAwIG4gCjAwMDAwMDAxODkgMDAwMDAgbiAKMDAwMDAwMDI0MCAwMDAwMCBuIAowMDAwMDAwMzY5IDAwMDAwIG4gCjAwMDAwMDA1ODUgMDAwMDAgbiAKMDAwMDAwMjA3OSAwMDAwMCBuIAp0cmFpbGVyCjw8L1Jvb3QgMiAwIFIvSURbKHtbPrtqnyTtl1Nt5AL9dhIpKHtbPrtqnyTtl1Nt5AL9dhIpXS9JbmZvIDcgMCBSL1NpemUgOD4+CnN0YXJ0eHJlZgoyMTU4CiUlRU9GCg=="

    # Create envelope using form-data
    form_data = {
        'name': 'Employment Contract - John Doe',
        'documents[0][base64]': test_pdf,
        'documents[0][name]': 'employment-contract.pdf',
        'recipients[0][name]': 'John Doe',
        'recipients[0][email]': 'john.doe@example.com',
        'recipients[0][order]': '1'
    }

    response = requests.post(
        'https://api.lexgo.cl/api/v1/envelopes',
        headers={'Authorization': 'YOUR_API_KEY'},
        data=form_data
    )

    envelope = response.json()['envelope']
    envelope_id = envelope['id']
    print(f"Created envelope: {envelope_id}")
    ```

=== "Node.js"
    ```javascript
    const fetch = require('node-fetch');
    const FormData = require('form-data');

    // Test PDF (or read from file: Buffer.from(fs.readFileSync('file.pdf')).toString('base64'))
    const testPdf = "JVBERi0xLjUKJc/s/+jXy80KMSAwIG9iago8PC9UeXBlL1BhZ2UvTWVkaWFCb3hbMCAwIDU5NS4yNzU1OTEgODQxLjg4OTc2NF0vUGFyZW50IDMgMCBSL1Jlc291cmNlczw8L0ZvbnQ8PC9GMSA2IDAgUj4+Pj4vQ29udGVudHMgNCAwIFI+PgplbmRvYmoKMiAwIG9iago8PC9UeXBlL0NhdGFsb2cvUGFnZXMgMyAwIFI+PgplbmRvYmoKMyAwIG9iago8PC9UeXBlL1BhZ2VzL0tpZHNbMSAwIFJdL0NvdW50IDE+PgplbmRvYmoKNCAwIG9iago8PC9GaWx0ZXIvRmxhdGVEZWNvZGUvTGVuZ3RoIDYzPj5zdHJlYW0KeNrTdzNUMDJQCEnjcgrhMlQwAEJDBRMDBXMLoGAuV7RGQFFpalKigk9qRXq+QnBmep5mbIgXl2sIFwCBuw6ACmVuZHN0cmVhbQplbmRvYmoKNSAwIG9iago8PC9UeXBlL0ZvbnREZXNjcmlwdG9yL0ZvbnROYW1lL0hlbHZldGljYS9Gb250V2VpZ2h0IDUwMC9Gb250QkJveFstMTY2LjAgLTIyNS4wIDEwMDAuMCA5MzEuMF0vSXRhbGljQW5nbGUgMC4wL0FzY2VudCA3MTguMC9EZXNjZW50IC0yMDcuMC9DYXBIZWlnaHQgNzE4LjAvWEhlaWdodCA1MjMuMC9TdGVtSCA3Ni4wL1N0ZW1WIDg4LjAvRmxhZ3MgMzI+PgplbmRvYmoKNiAwIG9iago8PC9UeXBlL0ZvbnQvU3VidHlwZS9UeXBlMS9CYXNlRm9udC9IZWx2ZXRpY2EvRm9udERlc2NyaXB0b3IgNSAwIFIvRmlyc3RDaGFyIDMyL0xhc3RDaGFyIDI1NS9XaWR0aHNbMjc4LjAgMjc4LjAgMzU1LjAgNTU2LjAgNTU2LjAgODg5LjAgNjY3LjAgMTkxLjAgMzMzLjAgMzMzLjAgMzg5LjAgNTg0LjAgMjc4LjAgMzMzLjAgMjc4LjAgMjc4LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgMjc4LjAgMjc4LjAgNTg0LjAgNTg0LjAgNTg0LjAgNTU2LjAgMTAxNS4wIDY2Ny4wIDY2Ny4wIDcyMi4wIDcyMi4wIDY2Ny4wIDYxMS4wIDc3OC4wIDcyMi4wIDI3OC4wIDUwMC4wIDY2Ny4wIDU1Ni4wIDgzMy4wIDcyMi4wIDc3OC4wIDY2Ny4wIDc3OC4wIDcyMi4wIDY2Ny4wIDYxMS4wIDcyMi4wIDY2Ny4wIDk0NC4wIDY2Ny4wIDY2Ny4wIDYxMS4wIDI3OC4wIDI3OC4wIDI3OC4wIDQ2OS4wIDU1Ni4wIDMzMy4wIDU1Ni4wIDU1Ni4wIDUwMC4wIDU1Ni4wIDU1Ni4wIDI3OC4wIDU1Ni4wIDU1Ni4wIDIyMi4wIDIyMi4wIDUwMC4wIDIyMi4wIDgzMy4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDMzMy4wIDUwMC4wIDI3OC4wIDU1Ni4wIDUwMC4wIDcyMi4wIDUwMC4wIDUwMC4wIDUwMC4wIDMzNC4wIDI2MC4wIDMzNC4wIDU4NC4wIDM1MC4wIDU1Ni4wIDM1MC4wIDIyMi4wIDU1Ni4wIDMzMy4wIDEwMDAuMCA1NTYuMCA1NTYuMCAzMzMuMCAxMDAwLjAgNjY3LjAgMzMzLjAgMTAwMC4wIDM1MC4wIDYxMS4wIDM1MC4wIDM1MC4wIDIyMi4wIDIyMi4wIDMzMy4wIDMzMy4wIDM1MC4wIDU1Ni4wIDEwMDAuMCAzMzMuMCAxMDAwLjAgNTAwLjAgMzMzLjAgOTQ0LjAgMzUwLjAgNTAwLjAgNjY3LjAgMjc4LjAgMzMzLjAgNTU2LjAgNTU2LjAgNTU2LjAgNTU2LjAgMjYwLjAgNTU2LjAgMzMzLjAgNzM3LjAgMzcwLjAgNTU2LjAgNTg0LjAgMzMzLjAgNzM3LjAgMzMzLjAgNDAwLjAgNTg0LjAgMzMzLjAgMzMzLjAgMzMzLjAgNTU2LjAgNTM3LjAgMjc4LjAgMzMzLjAgMzMzLjAgMzY1LjAgNTU2LjAgODM0LjAgODM0LjAgODM0LjAgNjExLjAgNjY3LjAgNjY3LjAgNjY3LjAgNjY3LjAgNjY3LjAgNjY3LjAgMTAwMC4wIDcyMi4wIDY2Ny4wIDY2Ny4wIDY2Ny4wIDY2Ny4wIDI3OC4wIDI3OC4wIDI3OC4wIDI3OC4wIDcyMi4wIDcyMi4wIDc3OC4wIDc3OC4wIDc3OC4wIDc3OC4wIDc3OC4wIDU4NC4wIDc3OC4wIDcyMi4wIDcyMi4wIDcyMi4wIDcyMi4wIDY2Ny4wIDY2Ny4wIDYxMS4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDg4OS4wIDUwMC4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDI3OC4wIDI3OC4wIDI3OC4wIDI3OC4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU4NC4wIDYxMS4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDU1Ni4wIDUwMC4wIDU1Ni4wIDUwMC4wXS9FbmNvZGluZy9XaW5BbnNpRW5jb2Rpbmc+PgplbmRvYmoKNyAwIG9iago8PC9Nb2REYXRlKEQ6MjAyNDExMjgxNjUxMDEpL1Byb2R1Y2VyKEhleGFQREYgdmVyc2lvbiAwLjQ2LjApPj4KZW5kb2JqCnhyZWYKMCA4CjAwMDAwMDAwMDAgNjU1MzUgZiAKMDAwMDAwMDAxOCAwMDAwMCBuIAowMDAwMDAwMTQ0IDAwMDAwIG4gCjAwMDAwMDAxODkgMDAwMDAgbiAKMDAwMDAwMDI0MCAwMDAwMCBuIAowMDAwMDAwMzY5IDAwMDAwIG4gCjAwMDAwMDA1ODUgMDAwMDAgbiAKMDAwMDAwMjA3OSAwMDAwMCBuIAp0cmFpbGVyCjw8L1Jvb3QgMiAwIFIvSURbKHtbPrtqnyTtl1Nt5AL9dhIpKHtbPrtqnyTtl1Nt5AL9dhIpXS9JbmZvIDcgMCBSL1NpemUgOD4+CnN0YXJ0eHJlZgoyMTU4CiUlRU9GCg==";

    // Create envelope using form-data
    const formData = new FormData();
    formData.append('name', 'Employment Contract - John Doe');
    formData.append('documents[0][base64]', testPdf);
    formData.append('documents[0][name]', 'employment-contract.pdf');
    formData.append('recipients[0][name]', 'John Doe');
    formData.append('recipients[0][email]', 'john.doe@example.com');
    formData.append('recipients[0][order]', '1');

    const response = await fetch('https://api.lexgo.cl/api/v1/envelopes', {
      method: 'POST',
      headers: {
        'Authorization': 'YOUR_API_KEY'
      },
      body: formData
    });

    const data = await response.json();
    const envelopeId = data.envelope.id;
    console.log(`Created envelope: ${envelopeId}`);
    ```

### Response

```json
{
  "envelope": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "Employment Contract - John Doe",
    "status": "CREATED",
    "envelope_signers": [
      {
        "id": "123e4567-e89b-12d3-a456-426614174000",
        "name": "John Doe",
        "email": "john.doe@example.com",
        "order": 1,
        "status": "PENDING"
      }
    ],
    "created_at": "2025-11-01T10:30:00Z"
  },
  "request_id": "req_abc123"
}
```

!!! success "Envelope Created"
    Your envelope is now created with status `CREATED`. Next, you'll send it to the recipient.

## Step 2: Send the Envelope

After creating the envelope, send it to the recipient to start the signature process.

=== "cURL"
    ```bash
    curl -X POST https://api.lexgo.cl/api/v1/envelopes/{envelope_id}/send_invitation \
      -H "Authorization: YOUR_API_KEY"
    ```

=== "Python"
    ```python
    response = requests.post(
        f'https://api.lexgo.cl/api/v1/envelopes/{envelope_id}/send_invitation',
        headers={'Authorization': 'YOUR_API_KEY'}
    )

    result = response.json()
    print(f"Envelope sent: {result['success']}")
    ```

=== "Node.js"
    ```javascript
    const response = await fetch(
      `https://api.lexgo.cl/api/v1/envelopes/${envelopeId}/send_invitation`,
      {
        method: 'POST',
        headers: {
          'Authorization': 'YOUR_API_KEY'
        }
      }
    );

    const result = await response.json();
    console.log(`Envelope sent: ${result.success}`);
    ```

### Response

```json
{
  "success": true,
  "request_id": "req_def456"
}
```

!!! success "Email Sent"
    The recipient will receive an email with a secure link to access and sign the document.

## Step 3: Track Envelope Status

Monitor the envelope status to know when it's been signed.

=== "cURL"
    ```bash
    curl https://api.lexgo.cl/api/v1/envelopes/{envelope_id} \
      -H "Authorization: YOUR_API_KEY"
    ```

=== "Python"
    ```python
    response = requests.get(
        f'https://api.lexgo.cl/api/v1/envelopes/{envelope_id}',
        headers={'Authorization': 'YOUR_API_KEY'}
    )

    envelope = response.json()['envelope']
    print(f"Status: {envelope['status']}")
    ```

=== "Node.js"
    ```javascript
    const response = await fetch(
      `https://api.lexgo.cl/api/v1/envelopes/${envelopeId}`,
      { headers: { 'Authorization': 'YOUR_API_KEY' } }
    );

    const envelope = (await response.json()).envelope;
    console.log(`Status: ${envelope.status}`);
    ```

### Envelope States

| Status | Description |
|--------|-------------|
| `CREATED` | Envelope created, not yet sent |
| `IN_PROGRESS` | Sent to recipients, awaiting signatures |
| `SUCCESS` | All recipients have signed |
| `VOIDED` | Envelope was cancelled |

## Step 4: Download Evidence Sheet

Once all recipients have signed (`status: SUCCESS`), download the evidence sheet for legal compliance.

=== "cURL"
    ```bash
    curl https://api.lexgo.cl/api/v1/envelopes/{envelope_id}/evidence \
      -H "Authorization: YOUR_API_KEY"
    ```

=== "Python"
    ```python
    response = requests.get(
        f'https://api.lexgo.cl/api/v1/envelopes/{envelope_id}/evidence',
        headers={'Authorization': 'YOUR_API_KEY'}
    )

    evidence_url = response.json()['evidence_document_url']
    print(f"Evidence sheet: {evidence_url}")
    ```

=== "Node.js"
    ```javascript
    const response = await fetch(
      `https://api.lexgo.cl/api/v1/envelopes/${envelopeId}/evidence`,
      { headers: { 'Authorization': 'YOUR_API_KEY' } }
    );

    const evidenceUrl = (await response.json()).evidence_document_url;
    console.log(`Evidence sheet: ${evidenceUrl}`);
    ```

### Response

```json
{
  "success": true,
  "evidence_document_url": "https://s3.amazonaws.com/lexgo-evidence/envelope_123_evidence.pdf",
  "request_id": "req_ghi789"
}
```

!!! info "Evidence Sheet"
    The evidence sheet contains the complete audit trail including timestamps, IP addresses, and cryptographic verification.

## Complete Example

Here's a complete end-to-end example in Python:

```python
import requests
import base64
import time

API_KEY = 'your_api_key_here'
BASE_URL = 'https://api.lexgo.cl/api/v1'

headers = {
    'Authorization': API_KEY,
    'Content-Type': 'application/json'
}

# 1. Create envelope
with open('contract.pdf', 'rb') as f:
    pdf_base64 = base64.b64encode(f.read()).decode('utf-8')

response = requests.post(
    f'{BASE_URL}/envelopes',
    headers=headers,
    json={
        'name': 'Employment Contract - John Doe',
        'envelope_signers': [
            {
                'name': 'John Doe',
                'email': 'john.doe@example.com',
                'order': 1
            }
        ],
        'files': [
            {
                'name': 'employment-contract.pdf',
                'content_base64': pdf_base64
            }
        ]
    }
)

envelope_id = response.json()['envelope']['id']
print(f"✓ Created envelope: {envelope_id}")

# 2. Send envelope
response = requests.put(
    f'{BASE_URL}/envelopes/{envelope_id}/send',
    headers=headers
)

print(f"✓ Sent envelope to recipient")

# 3. Poll for completion
while True:
    response = requests.get(
        f'{BASE_URL}/envelopes/{envelope_id}',
        headers=headers
    )

    status = response.json()['envelope']['status']
    print(f"  Status: {status}")

    if status == 'SUCCESS':
        break

    time.sleep(30)  # Check every 30 seconds

# 4. Download evidence
response = requests.get(
    f'{BASE_URL}/envelopes/{envelope_id}/evidence',
    headers=headers
)

evidence_url = response.json()['evidence_document_url']
print(f"✓ Evidence sheet: {evidence_url}")
```

## Next Steps

Now that you've created your first envelope, explore more features:

- [Enable email validation (2FA)](../guides/email-validation.md) for enhanced security
- [Set up webhooks](../guides/webhook-integration.md) for real-time notifications
- [Customize email templates](../guides/email-templates.md) with your branding
- [Add multiple signers](../api/envelopes.md#multiple-signers) with sequential workflow

## Common Issues

### PDF Encoding Error
Make sure to encode your PDF in base64 format:

```python
import base64
with open('file.pdf', 'rb') as f:
    content = base64.b64encode(f.read()).decode('utf-8')
```

### 401 Unauthorized
Verify your API key is correct and included in the Authorization header.

### Envelope Not Found
Ensure you're using the correct API key (Sandbox vs Live) and that the envelope was created with that key.

## Support

Need help? [Contact us](https://www.lexgo.cl/contacto) or check our [API reference](../api/overview.md) for detailed documentation.
