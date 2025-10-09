# Cache Endpoints

Complete reference for Vectorcache API endpoints.

## Query Cache

Query the semantic cache for a response.

### Endpoint

```
POST /v1/cache/query
```

### Request

#### Headers

| Header | Value | Required |
|--------|-------|----------|
| `Authorization` | `Bearer YOUR_API_KEY` | Yes |
| `Content-Type` | `application/json` | Yes |

#### Body Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `prompt` | string | Yes | - | The text prompt to cache/query |
| `model` | string | Yes | - | LLM model identifier |
| `similarity_threshold` | number | No | `0.85` | Minimum similarity score (0-1) |
| `context` | string | No | `null` | Additional context for segmentation |
| `include_debug` | boolean | No | `false` | Include debug information |

#### Example Request

```json
{
  "prompt": "What is machine learning?",
  "model": "gpt-4o",
  "similarity_threshold": 0.85,
  "context": "educational-content",
  "include_debug": false
}
```

### Response

#### Success Response (200 OK)

##### Cache Hit

```json
{
  "cache_hit": true,
  "response": "Machine learning is a subset of artificial intelligence...",
  "similarity_score": 0.92,
  "cost_saved": 0.003,
  "llm_provider": "cache"
}
```

##### Cache Miss

```json
{
  "cache_hit": false,
  "response": "Machine learning is a subset of artificial intelligence...",
  "similarity_score": null,
  "cost_saved": 0,
  "llm_provider": "openai"
}
```

#### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `cache_hit` | boolean | Whether the query matched a cached entry |
| `response` | string | The LLM response text |
| `similarity_score` | number \| null | Cosine similarity score (0-1), null on cache miss |
| `cost_saved` | number | Estimated cost saved in USD (0 on cache miss) |
| `llm_provider` | string | Source of response ('cache' or LLM provider name) |
| `debug` | object | Debug information (only if `include_debug: true`) |

#### Debug Information

When `include_debug: true`:

```json
{
  "cache_hit": true,
  "response": "...",
  "similarity_score": 0.92,
  "cost_saved": 0.003,
  "llm_provider": "cache",
  "debug": {
    "embedding_time_ms": 45,
    "search_time_ms": 12,
    "total_time_ms": 57,
    "matched_cache_entry_id": "uuid-here",
    "cache_entry_count": 1523
  }
}
```

### Error Responses

#### 400 Bad Request

Invalid request parameters:

```json
{
  "detail": "similarity_threshold must be between 0 and 1"
}
```

**Common causes:**
- Missing required fields (`prompt`, `model`)
- Invalid `similarity_threshold` (not between 0-1)
- Invalid JSON format

#### 401 Unauthorized

Authentication failed:

```json
{
  "detail": "Invalid or missing API key"
}
```

**Causes:**
- Missing `Authorization` header
- Invalid API key
- Revoked API key

#### 429 Too Many Requests

Rate limit exceeded:

```json
{
  "detail": "Rate limit exceeded. Please try again later."
}
```

**Headers included:**
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1640000000
```

#### 500 Internal Server Error

Server error:

```json
{
  "detail": "Internal server error. Please try again or contact support."
}
```

## Examples

### Basic Query

```bash
curl -X POST "https://api.vectorcache.com/v1/cache/query" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "What is AI?",
    "model": "gpt-4o"
  }'
```

### With Context

```bash
curl -X POST "https://api.vectorcache.com/v1/cache/query" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Explain neural networks",
    "context": "technical-documentation",
    "model": "gpt-4o",
    "similarity_threshold": 0.90
  }'
```

### With Debug Info

```bash
curl -X POST "https://api.vectorcache.com/v1/cache/query" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "What is deep learning?",
    "model": "gpt-4o",
    "include_debug": true
  }'
```

## Supported Models

Vectorcache supports all major LLM providers:

### OpenAI

- `gpt-4o`
- `gpt-4o-mini`
- `gpt-4-turbo`
- `gpt-3.5-turbo`

### Anthropic

- `claude-3-5-sonnet-20241022`
- `claude-3-5-haiku-20241022`
- `claude-3-opus-20240229`

### Google

- `gemini-1.5-pro`
- `gemini-1.5-flash`

### Other Providers

Check the [dashboard](https://app.vectorcache.com) for your configured LLM providers.

## Context-Based Segmentation

Use the `context` parameter to segment your cache by use case:

```javascript
// Educational content cache
await client.query({
  prompt: 'What is photosynthesis?',
  context: 'education-biology',
  model: 'gpt-4o'
});

// Technical documentation cache
await client.query({
  prompt: 'What is photosynthesis?',
  context: 'scientific-research',
  model: 'gpt-4o'
});
```

Even with the same prompt, these will be cached separately due to different contexts.

## Similarity Threshold

The `similarity_threshold` parameter controls cache sensitivity:

| Threshold | Behavior | Use Case |
|-----------|----------|----------|
| 0.95-1.0 | Very strict | Legal, medical, financial content |
| 0.85-0.94 | **Recommended** | General purpose, customer support |
| 0.75-0.84 | Relaxed | Educational content, FAQs |
| <0.75 | Very relaxed | Not recommended (low accuracy) |

### Example: Testing Thresholds

```javascript
// Strict - only nearly identical queries match
const strict = await client.query({
  prompt: 'What is machine learning?',
  model: 'gpt-4o',
  similarityThreshold: 0.95
});

// Relaxed - more cache hits, less precision
const relaxed = await client.query({
  prompt: 'What is machine learning?',
  model: 'gpt-4o',
  similarityThreshold: 0.80
});
```

## Cost Calculation

The `cost_saved` field estimates the LLM API cost you saved from the cache hit:

```json
{
  "cache_hit": true,
  "cost_saved": 0.003,  // $0.003 saved
  "llm_provider": "cache"
}
```

**Calculation:**
- Based on the model's input/output token pricing
- Includes both prompt and response tokens
- Updated automatically with latest pricing

**Example savings:**
- `gpt-4o` query: ~$0.002-0.005 per query saved
- 1,000 cache hits/day: ~$2-5 saved/day
- Annual savings: ~$730-1,825

## Performance

### Response Times

| Scenario | Typical Response Time |
|----------|----------------------|
| Cache Hit | 50-150ms |
| Cache Miss (with LLM call) | 1-5 seconds |
| Debug Mode | +10-20ms |

### Optimization Tips

1. **Use appropriate thresholds** - Higher thresholds = faster searches
2. **Enable caching** - First query is slow, subsequent ones are fast
3. **Batch similar queries** - Group related prompts together
4. **Monitor debug metrics** - Use `include_debug` to optimize

## Rate Limits

| Tier | Requests/Minute | Burst |
|------|----------------|-------|
| Free | 100 | 120 |
| Pro | 1,000 | 1,200 |
| Enterprise | Custom | Custom |

**Rate limit headers:**
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640000000
```

## Best Practices

1. **Always handle both cache hit and miss** - Your application should work in both scenarios
2. **Use context for segmentation** - Separate caches by use case
3. **Monitor similarity scores** - Tune thresholds based on actual scores
4. **Implement retry logic** - Handle 429 errors with exponential backoff
5. **Track cost savings** - Monitor `cost_saved` to measure ROI

## Next Steps

- [Error Handling](errors.md) - Handle API errors
- [Best Practices](../guides/best-practices.md) - Production tips
- [Similarity Tuning](../guides/similarity-tuning.md) - Optimize cache hits
