# Installation

Install the Vectorcache SDK for your preferred language.

## JavaScript/TypeScript

### NPM

```bash
npm install vectorcache-js
```

### Yarn

```bash
yarn add vectorcache-js
```

### pnpm

```bash
pnpm add vectorcache-js
```

### Requirements

- Node.js 16.x or higher
- TypeScript 4.5+ (if using TypeScript)

## Python

### pip

```bash
pip install vectorcache-python
```

### Poetry

```bash
poetry add vectorcache-python
```

### Requirements

- Python 3.8 or higher

## Direct API (No SDK)

You can also use Vectorcache without an SDK by making direct HTTP requests to our REST API.

### cURL

No installation needed - just use cURL:

```bash
curl -X POST "https://api.vectorcache.com/v1/cache/query" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Your prompt here",
    "model": "gpt-4o",
    "similarity_threshold": 0.85
  }'
```

### HTTP Clients

Any HTTP client library works:

=== "JavaScript (Axios)"

    ```javascript
    const axios = require('axios');

    const response = await axios.post(
      'https://api.vectorcache.com/v1/cache/query',
      {
        prompt: 'Your prompt here',
        model: 'gpt-4o',
        similarity_threshold: 0.85
      },
      {
        headers: {
          'Authorization': `Bearer ${apiKey}`,
          'Content-Type': 'application/json'
        }
      }
    );
    ```

=== "Python (httpx)"

    ```python
    import httpx

    async with httpx.AsyncClient() as client:
        response = await client.post(
            'https://api.vectorcache.com/v1/cache/query',
            json={
                'prompt': 'Your prompt here',
                'model': 'gpt-4o',
                'similarity_threshold': 0.85
            },
            headers={
                'Authorization': f'Bearer {api_key}',
                'Content-Type': 'application/json'
            }
        )
        result = response.json()
    ```

## Verify Installation

=== "JavaScript/TypeScript"

    ```javascript
    const { VectorcacheClient } = require('vectorcache-js');
    console.log('Vectorcache SDK installed successfully!');
    ```

=== "Python"

    ```python
    import vectorcache
    print('Vectorcache SDK installed successfully!')
    ```

## Next Steps

- [Quick Start Guide](quickstart.md) - Make your first request
- [Configuration](configuration.md) - Configure the SDK
- [API Reference](../api/overview.md) - Explore all endpoints
