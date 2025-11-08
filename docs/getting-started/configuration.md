# Configuration

Configure the Vectorcache SDK for your application.

## JavaScript/TypeScript Configuration

### Basic Setup

```typescript
import { VectorcacheClient } from 'vectorcache';

const client = new VectorcacheClient({
  apiKey: 'your_api_key_here',
  baseUrl: 'https://api.vectorcache.ai' // optional, defaults to production
});
```

### Configuration Options

| Option | Type | Required | Default | Description |
|--------|------|----------|---------|-------------|
| `apiKey` | string | Yes | - | Your Vectorcache API key |
| `baseUrl` | string | No | `https://api.vectorcache.ai` | API base URL |
| `timeout` | number | No | `30000` | Request timeout in milliseconds |

### Environment Variables

Store your API key in environment variables:

```typescript
const client = new VectorcacheClient({
  apiKey: process.env.VECTORCACHE_API_KEY!,
});
```

**`.env` file:**
```bash
VECTORCACHE_API_KEY=your_api_key_here
```

### TypeScript Types

The SDK is fully typed. Import types as needed:

```typescript
import {
  VectorcacheClient,
  CacheQueryRequest,
  CacheQueryResponse
} from 'vectorcache';

const request: CacheQueryRequest = {
  prompt: 'What is AI?',
  model: 'gpt-4o',
  similarityThreshold: 0.85
};

const response: CacheQueryResponse = await client.query(request);
```

## Python Configuration

### Basic Setup

```python
import requests
import os

api_key = os.environ.get('VECTORCACHE_API_KEY')
base_url = 'https://api.vectorcache.ai'

headers = {
    'Authorization': f'Bearer {api_key}',
    'Content-Type': 'application/json'
}
```

### Environment Variables

**`.env` file:**
```bash
VECTORCACHE_API_KEY=your_api_key_here
```

**Using python-dotenv:**
```python
from dotenv import load_dotenv
import os

load_dotenv()

api_key = os.environ.get('VECTORCACHE_API_KEY')
```

## Query Parameters

### Required Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `prompt` | string | The text prompt to cache/query |
| `model` | string | LLM model identifier (e.g., 'gpt-4o', 'claude-3-5-sonnet-20241022') |

### Optional Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `similarity_threshold` | number | `0.85` | Minimum similarity score (0-1) for cache hits |
| `context` | string | `null` | Additional context for the query |
| `project_id` | string | Auto-detected | Project ID (inferred from API key) |
| `include_debug` | boolean | `false` | Include debug information in response |

### Example with All Parameters

```typescript
const result = await client.query({
  prompt: 'Explain machine learning',
  context: 'Educational content for beginners',
  model: 'gpt-4o',
  similarityThreshold: 0.85,
  includeDebug: true
});
```

## Similarity Threshold

The `similarity_threshold` parameter controls cache sensitivity:

- **0.95-1.0**: Very strict - only nearly identical queries match
- **0.85-0.94**: Recommended - good balance of accuracy and hit rate
- **0.70-0.84**: Relaxed - more cache hits but less precise matches
- **Below 0.70**: Not recommended - may return irrelevant cached responses

### Finding the Right Threshold

Start with `0.85` and adjust based on your use case:

```typescript
// Educational content - can be more relaxed
const eduResult = await client.query({
  prompt: 'What is photosynthesis?',
  similarityThreshold: 0.80 // Lower threshold OK
});

// Legal/Medical - needs precision
const legalResult = await client.query({
  prompt: 'Interpret contract clause 5.2',
  similarityThreshold: 0.92 // Higher threshold for accuracy
});
```

Learn more in the [Similarity Tuning Guide](../guides/similarity-tuning.md).

## Error Handling

### JavaScript/TypeScript

```typescript
try {
  const result = await client.query({
    prompt: 'What is AI?',
    model: 'gpt-4o'
  });
  console.log(result.response);
} catch (error) {
  if (error.response?.status === 401) {
    console.error('Invalid API key');
  } else if (error.response?.status === 429) {
    console.error('Rate limit exceeded');
  } else {
    console.error('Unexpected error:', error.message);
  }
}
```

### Python

```python
try:
    response = requests.post(
        f"{base_url}/v1/cache/query",
        json=data,
        headers=headers
    )
    response.raise_for_status()
    result = response.json()
except requests.exceptions.HTTPError as e:
    if e.response.status_code == 401:
        print("Invalid API key")
    elif e.response.status_code == 429:
        print("Rate limit exceeded")
    else:
        print(f"HTTP error: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
```

See [Error Handling](../api/errors.md) for complete error codes.

## Production Best Practices

### 1. Use Environment Variables

Never hardcode API keys:

```typescript
// ❌ Bad
const client = new VectorcacheClient({
  apiKey: 'vc_1234567890abcdef'
});

// ✅ Good
const client = new VectorcacheClient({
  apiKey: process.env.VECTORCACHE_API_KEY!
});
```

### 2. Set Appropriate Timeouts

```typescript
const client = new VectorcacheClient({
  apiKey: process.env.VECTORCACHE_API_KEY!,
  timeout: 10000 // 10 seconds for production
});
```

### 3. Handle Errors Gracefully

Always have fallback logic:

```typescript
async function getResponse(prompt: string) {
  try {
    const result = await client.query({ prompt, model: 'gpt-4o' });
    return result.response;
  } catch (error) {
    console.error('Vectorcache error:', error);
    // Fallback to direct LLM call
    return await fallbackLLMCall(prompt);
  }
}
```

### 4. Monitor Performance

Track cache performance in your application:

```typescript
const result = await client.query({ prompt, model: 'gpt-4o' });

// Log metrics
analytics.track('vectorcache_query', {
  cache_hit: result.cache_hit,
  similarity_score: result.similarity_score,
  cost_saved: result.cost_saved
});
```

## Next Steps

- [API Reference](../api/overview.md) - Complete API documentation
- [Best Practices](../guides/best-practices.md) - Production tips
- [Similarity Tuning](../guides/similarity-tuning.md) - Optimize cache hit rates
