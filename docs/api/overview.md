# API Overview

Vectorcache provides a RESTful API for semantic caching of LLM responses.

## Base URL

```
https://api.vectorcache.com
```

## Authentication

All API requests require authentication using an API key in the Authorization header:

```
Authorization: Bearer YOUR_API_KEY
```

Get your API key from the [dashboard](https://app.vectorcache.com).

## Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/v1/cache/query` | Query the semantic cache |

## Request Format

All requests must include:

- `Content-Type: application/json` header
- JSON request body with required parameters
- Bearer token authentication

## Response Format

All successful responses return JSON with:

```json
{
  "cache_hit": boolean,
  "response": string,
  "similarity_score": number | null,
  "cost_saved": number,
  "llm_provider": string
}
```

## Rate Limits

- **Free tier**: 100 requests/minute
- **Pro tier**: 1,000 requests/minute
- **Enterprise**: Custom limits

Rate limit headers are included in responses:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640000000
```

## Versioning

The API is versioned via the URL path (`/v1/`). Breaking changes will result in a new version.

Current version: **v1**

## Error Handling

All errors return a JSON response with a `detail` field:

```json
{
  "detail": "Error message describing what went wrong"
}
```

See [Error Handling](errors.md) for complete error codes and handling.

## Quick Example

```bash
curl -X POST "https://api.vectorcache.com/v1/cache/query" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "What is machine learning?",
    "model": "gpt-4o",
    "similarity_threshold": 0.85
  }'
```

## Next Steps

- [Authentication](authentication.md) - API key management
- [Cache Endpoints](cache.md) - Detailed endpoint documentation
- [Error Handling](errors.md) - Error codes and handling
