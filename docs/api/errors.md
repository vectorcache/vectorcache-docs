# Error Handling

Complete guide to handling errors in the Vectorcache API.

## Error Response Format

All errors return JSON with a `detail` field:

```json
{
  "detail": "Error message describing what went wrong"
}
```

## HTTP Status Codes

| Code | Status | Description |
|------|--------|-------------|
| 200 | OK | Request successful |
| 400 | Bad Request | Invalid request parameters |
| 401 | Unauthorized | Authentication failed |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource not found |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error |
| 502 | Bad Gateway | LLM provider error |
| 503 | Service Unavailable | Service temporarily unavailable |

## Common Errors

### 400 Bad Request

#### Missing Required Fields

```json
{
  "detail": "Field required: prompt"
}
```

**Cause:** Missing `prompt` or `model` in request

**Solution:**
```javascript
// ❌ Missing model
{ prompt: "What is AI?" }

// ✅ Include all required fields
{ prompt: "What is AI?", model: "gpt-4o" }
```

#### Invalid Similarity Threshold

```json
{
  "detail": "similarity_threshold must be between 0 and 1"
}
```

**Cause:** `similarity_threshold` outside valid range

**Solution:**
```javascript
// ❌ Invalid threshold
{ similarity_threshold: 1.5 }

// ✅ Valid threshold
{ similarity_threshold: 0.85 }
```

#### Invalid JSON

```json
{
  "detail": "Invalid JSON format"
}
```

**Cause:** Malformed JSON in request body

**Solution:** Validate JSON before sending

### 401 Unauthorized

#### Missing API Key

```json
{
  "detail": "Invalid or missing API key"
}
```

**Cause:** No `Authorization` header

**Solution:**
```javascript
// ❌ Missing auth header
fetch(url, {
  method: 'POST',
  body: JSON.stringify(data)
});

// ✅ Include auth header
fetch(url, {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${apiKey}`
  },
  body: JSON.stringify(data)
});
```

#### Invalid API Key

```json
{
  "detail": "Invalid API key"
}
```

**Cause:** API key is invalid or revoked

**Solution:**
- Verify your API key in the dashboard
- Check if the key has been revoked
- Create a new API key if needed

### 403 Forbidden

#### Insufficient Permissions

```json
{
  "detail": "API key does not have permission to access this project"
}
```

**Cause:** Using an API key from a different project

**Solution:** Use the correct API key for this project

### 429 Too Many Requests

#### Rate Limit Exceeded

```json
{
  "detail": "Rate limit exceeded. Please try again later."
}
```

**Headers:**
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1640000000
```

**Solution:** Implement retry with exponential backoff

```javascript
async function queryWithRetry(client, request, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await client.query(request);
    } catch (error) {
      if (error.statusCode === 429) {
        const resetTime = error.headers['x-ratelimit-reset'];
        const waitTime = Math.max(
          (resetTime * 1000) - Date.now(),
          1000 * Math.pow(2, attempt)
        );
        await new Promise(resolve => setTimeout(resolve, waitTime));
        continue;
      }
      throw error;
    }
  }
}
```

### 500 Internal Server Error

#### Server Error

```json
{
  "detail": "Internal server error. Please try again or contact support."
}
```

**Cause:** Unexpected server error

**Solution:**
- Retry the request
- If persists, contact support with request details

### 502 Bad Gateway

#### LLM Provider Error

```json
{
  "detail": "LLM provider error: Invalid API key"
}
```

**Cause:** Issue with your LLM provider API key

**Solution:**
- Verify your LLM API key in Settings → LLM Keys
- Check if you have sufficient credits with the provider
- Ensure the model is available for your LLM provider

### 503 Service Unavailable

#### Service Temporarily Unavailable

```json
{
  "detail": "Service temporarily unavailable"
}
```

**Cause:** Service maintenance or high load

**Solution:** Retry with exponential backoff

## Error Handling Patterns

### JavaScript/TypeScript

```typescript
import { VectorcacheClient, VectorcacheError } from 'vectorcache-js';

const client = new VectorcacheClient({ apiKey: 'YOUR_API_KEY' });

try {
  const result = await client.query({
    prompt: 'What is AI?',
    model: 'gpt-4o'
  });

  console.log(result.response);

} catch (error) {
  if (error instanceof VectorcacheError) {
    switch (error.statusCode) {
      case 400:
        console.error('Invalid request:', error.message);
        break;
      case 401:
        console.error('Authentication failed - check your API key');
        break;
      case 429:
        console.error('Rate limit exceeded - please slow down');
        break;
      case 500:
        console.error('Server error - retrying...');
        // Implement retry logic
        break;
      case 502:
        console.error('LLM provider error:', error.message);
        break;
      default:
        console.error('Unexpected error:', error.message);
    }
  } else {
    console.error('Network or unknown error:', error);
  }
}
```

### Python

```python
import requests
from requests.exceptions import HTTPError, Timeout, RequestException

def query_with_error_handling(prompt: str, model: str, api_key: str):
    url = 'https://api.vectorcache.com/v1/cache/query'

    headers = {
        'Authorization': f'Bearer {api_key}',
        'Content-Type': 'application/json'
    }

    data = {
        'prompt': prompt,
        'model': model
    }

    try:
        response = requests.post(url, json=data, headers=headers, timeout=30)
        response.raise_for_status()
        return response.json()

    except HTTPError as e:
        status_code = e.response.status_code
        error_detail = e.response.json().get('detail', 'Unknown error')

        if status_code == 400:
            raise ValueError(f"Invalid request: {error_detail}")
        elif status_code == 401:
            raise ValueError("Authentication failed - check your API key")
        elif status_code == 429:
            raise ValueError("Rate limit exceeded - please slow down")
        elif status_code == 500:
            raise ValueError(f"Server error: {error_detail}")
        elif status_code == 502:
            raise ValueError(f"LLM provider error: {error_detail}")
        else:
            raise ValueError(f"HTTP error {status_code}: {error_detail}")

    except Timeout:
        raise ValueError("Request timed out")

    except RequestException as e:
        raise ValueError(f"Request failed: {e}")
```

## Retry Strategies

### Exponential Backoff

```javascript
async function exponentialBackoff(
  fn,
  maxRetries = 3,
  baseDelay = 1000
) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      // Don't retry on client errors (4xx except 429)
      if (error.statusCode >= 400 &&
          error.statusCode < 500 &&
          error.statusCode !== 429) {
        throw error;
      }

      if (attempt === maxRetries) {
        throw error;
      }

      const delay = Math.min(baseDelay * Math.pow(2, attempt - 1), 10000);
      console.log(`Retry attempt ${attempt} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// Usage
const result = await exponentialBackoff(
  () => client.query({ prompt: 'What is AI?', model: 'gpt-4o' })
);
```

### Rate Limit Aware Retry

```python
import time
from datetime import datetime

def retry_with_rate_limit(fn, max_retries=3):
    for attempt in range(1, max_retries + 1):
        try:
            return fn()
        except requests.exceptions.HTTPError as e:
            if e.response.status_code == 429:
                # Check rate limit reset time
                reset_time = int(e.response.headers.get('X-RateLimit-Reset', 0))
                if reset_time:
                    wait_time = max(reset_time - int(time.time()), 0) + 1
                else:
                    wait_time = min(2 ** attempt, 60)

                print(f"Rate limited. Waiting {wait_time}s...")
                time.sleep(wait_time)
                continue
            raise
        except Exception as e:
            if attempt == max_retries:
                raise
            delay = min(2 ** attempt, 10)
            print(f"Retry {attempt} after {delay}s")
            time.sleep(delay)
```

## Validation Best Practices

### Client-Side Validation

Validate inputs before making API calls:

```typescript
function validateQueryRequest(request: CacheQueryRequest): void {
  if (!request.prompt || request.prompt.trim().length === 0) {
    throw new Error('Prompt is required');
  }

  if (!request.model || request.model.trim().length === 0) {
    throw new Error('Model is required');
  }

  if (request.similarityThreshold !== undefined) {
    if (request.similarityThreshold < 0 || request.similarityThreshold > 1) {
      throw new Error('Similarity threshold must be between 0 and 1');
    }
  }
}

// Usage
try {
  validateQueryRequest(request);
  const result = await client.query(request);
} catch (error) {
  console.error('Validation error:', error.message);
}
```

### Graceful Degradation

Implement fallback logic when Vectorcache is unavailable:

```typescript
async function queryWithFallback(prompt: string, model: string) {
  try {
    // Try Vectorcache first
    const result = await client.query({ prompt, model });
    return result.response;
  } catch (error) {
    console.warn('Vectorcache unavailable, falling back to direct LLM');
    // Fallback to direct LLM call
    return await directLLMCall(prompt, model);
  }
}
```

## Monitoring and Logging

### Log Error Details

```typescript
function logError(error: VectorcacheError, context: any) {
  const errorLog = {
    timestamp: new Date().toISOString(),
    statusCode: error.statusCode,
    message: error.message,
    context: context,
    headers: error.headers
  };

  // Send to logging service
  logger.error('Vectorcache API error', errorLog);

  // For 5xx errors, alert operations team
  if (error.statusCode >= 500) {
    alertOps(errorLog);
  }
}
```

### Track Error Rates

```typescript
const errorMetrics = {
  total: 0,
  byStatusCode: {} as Record<number, number>
};

function trackError(error: VectorcacheError) {
  errorMetrics.total++;
  errorMetrics.byStatusCode[error.statusCode] =
    (errorMetrics.byStatusCode[error.statusCode] || 0) + 1;

  // Alert if error rate is high
  const errorRate = errorMetrics.total / totalRequests;
  if (errorRate > 0.05) { // 5% error rate
    alertHighErrorRate(errorMetrics);
  }
}
```

## Next Steps

- [Best Practices](../guides/best-practices.md) - Production deployment tips
- [API Reference](overview.md) - Complete API documentation
- [Support](../about/support.md) - Get help with errors
