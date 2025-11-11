# Authentication

## Overview

The LexgoSign API uses **API keys** for authentication. All API requests must include a valid API key in the `Authorization` header.

!!! warning "Security"
    Keep your API keys secure and never share them publicly. All requests must be made over HTTPS.

## Getting Your API Key

1. Log in to your Lexgo account at [https://app.lexgo.cl](https://app.lexgo.cl)
2. Navigate to **Settings** → **API Keys**
3. Click **Generate New API Key**
4. Copy and securely store your API key

!!! danger "Important"
    API keys are only shown once. If you lose your key, you'll need to generate a new one.

## Authentication Header

Include your API key in the `Authorization` header of every request:

```
Authorization: YOUR_API_KEY_HERE
```

## Example Requests

=== "cURL"
    ```bash
    curl https://api.lexgo.cl/api/v1/envelopes \
      -H "Authorization: your_api_key_here" \
      -H "Content-Type: application/json"
    ```

=== "Python"
    ```python
    import requests

    headers = {
        'Authorization': 'your_api_key_here',
        'Content-Type': 'application/json'
    }

    response = requests.get(
        'https://api.lexgo.cl/api/v1/envelopes',
        headers=headers
    )
    ```

=== "Node.js"
    ```javascript
    const fetch = require('node-fetch');

    const headers = {
        'Authorization': 'your_api_key_here',
        'Content-Type': 'application/json'
    };

    fetch('https://api.lexgo.cl/api/v1/envelopes', { headers })
        .then(res => res.json())
        .then(data => console.log(data));
    ```

=== "Ruby"
    ```ruby
    require 'net/http'
    require 'json'

    uri = URI('https://api.lexgo.cl/api/v1/envelopes')
    headers = {
      'Authorization' => 'your_api_key_here',
      'Content-Type' => 'application/json'
    }

    response = Net::HTTP.get_response(uri, headers)
    ```

## Testing Your Setup

After getting your API key, test it with the test endpoint:

**Endpoint**: `POST /api/v1/test`

=== "cURL"
    ```bash
    curl -X POST https://api.lexgo.cl/api/v1/test \
      -H "Authorization: your_api_key_here" \
      -F "test_param=Sample value 123"
    ```

=== "Python"
    ```python
    import requests

    response = requests.post(
        'https://api.lexgo.cl/api/v1/test',
        headers={'Authorization': 'your_api_key_here'},
        data={'test_param': 'Sample value 123'}
    )

    print(response.json())
    # Output: {
    #   "enterprise_name": "Your Company",
    #   "environment": "sandbox",
    #   "test_param": "Sample value 123"
    # }
    ```

=== "Node.js"
    ```javascript
    const fetch = require('node-fetch');
    const FormData = require('form-data');

    const form = new FormData();
    form.append('test_param', 'Sample value 123');

    fetch('https://api.lexgo.cl/api/v1/test', {
      method: 'POST',
      headers: { 'Authorization': 'your_api_key_here' },
      body: form
    })
      .then(res => res.json())
      .then(data => console.log(data));
    ```

**Response**:

```json
{
  "enterprise_name": "Your Company Name",
  "environment": "sandbox",
  "test_param": "Sample value 123"
}
```

**Response Fields**:
- `enterprise_name`: Your company/account name
- `environment`: Either `sandbox` or `live`
- `test_param`: Echoes back any parameter you sent (optional)

!!! tip "Quick Validation"
    Use this endpoint to verify your API key works before integrating other endpoints.

## Authentication Errors

### Invalid API Key

**Status Code**: `401 Unauthorized`

```json
{
  "error": "Invalid Api Key (key not found)"
}
```

**Causes**:
- API key doesn't exist
- API key has been deleted
- Typo in the Authorization header

### Missing Authorization Header

**Status Code**: `401 Unauthorized`

```json
{
  "error": "Missing Authorization Header"
}
```

**Causes**:
- `Authorization` header not included in request
- Header name misspelled

### Deactivated API Key

**Status Code**: `401 Unauthorized`

```json
{
  "error": "Invalid Api Key (key deactivated)"
}
```

**Causes**:
- API key has been deactivated in settings
- Enterprise account is suspended

## API Key Management

### Multiple Keys
You can create multiple API keys for different purposes:

- **Sandbox**: Test integration without legal validity (documents are watermarked)
- **Live**: Production use with legally binding signatures
- **Per-Application**: Create separate keys for different applications or integrations

### Key Rotation
Regularly rotate your API keys for security:

1. Generate a new API key
2. Update your application to use the new key
3. Verify the new key works correctly
4. Deactivate the old key

!!! tip "Best Practice"
    Rotate API keys every 90 days and immediately if compromised.

### Revoking Keys
To revoke an API key:

1. Go to **Settings** → **API Keys**
2. Find the key you want to revoke
3. Click **Deactivate**

!!! warning "Immediate Effect"
    Deactivating a key takes effect immediately. All requests using that key will fail.

## Security Best Practices

### Store Securely
- **Use environment variables**: Never hardcode API keys
- **Encrypt at rest**: Store keys encrypted in databases
- **Restrict access**: Limit who can view API keys

### Network Security
- **HTTPS only**: All requests must use HTTPS
- **IP whitelisting**: Restrict API access to known IP addresses
- **Rate limiting**: Implement client-side rate limiting

### Monitoring
- **Log requests**: Track API usage for anomaly detection
- **Alert on errors**: Monitor 401 errors for potential security issues
- **Audit access**: Review API key usage regularly

## Example: Storing API Keys

### Environment Variables

=== "Bash"
    ```bash
    export LEXGO_API_KEY="your_api_key_here"
    ```

=== "Python (.env)"
    ```python
    # .env file
    LEXGO_API_KEY=your_api_key_here

    # app.py
    import os
    from dotenv import load_dotenv

    load_dotenv()
    api_key = os.getenv('LEXGO_API_KEY')
    ```

=== "Node.js (.env)"
    ```javascript
    // .env file
    LEXGO_API_KEY=your_api_key_here

    // app.js
    require('dotenv').config();
    const apiKey = process.env.LEXGO_API_KEY;
    ```

=== "Ruby"
    ```ruby
    # .env file
    LEXGO_API_KEY=your_api_key_here

    # app.rb
    require 'dotenv/load'
    api_key = ENV['LEXGO_API_KEY']
    ```

## Next Steps

- [Quick Start Guide](quick-start.md) - Create your first envelope
- [API Overview](../api/overview.md) - Explore available endpoints
- [Error Codes](../api/errors.md) - Handle errors gracefully
