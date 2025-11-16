# Error Handling Best Practices

Learn how to properly handle errors when integrating Vectorcache into your application to ensure resilient, production-ready implementations.

## Overview

Vectorcache returns standard HTTP status codes to indicate the success or failure of API requests. Your application should implement proper error handling to gracefully handle failures and provide fallback behavior when the cache service is unavailable.

## HTTP Status Codes

Vectorcache uses the following HTTP status codes:

| Status Code | Meaning | When It Happens | What To Do |
|------------|---------|-----------------|------------|
| **200** | Success | Request processed successfully | Use the response data |
| **400** | Bad Request | Malformed request body or invalid parameters | Fix the request format |
| **401** | Unauthorized | Invalid or missing API key | Check your API key |
| **422** | Unprocessable Entity | Content not cacheable (images, PDFs, streaming, etc.) | Call LLM directly |
| **429** | Too Many Requests | Rate limit exceeded (monthly quota) | Wait or upgrade plan |
| **500** | Internal Server Error | Unexpected server error | Retry with exponential backoff |
| **503** | Service Unavailable | Database or service outage | Call LLM directly, check Retry-After header |

## Error Response Format

All error responses follow this structure:

```json
{
  "error": "Short error type",
  "message": "Human-readable error description",
  "retry_after": 60,  // (Optional) Seconds to wait before retry
  "fallback": "Suggested fallback action"  // (Optional)
}
```

## JavaScript/TypeScript Error Handling

### Complete Example with Fallback

```typescript
import { VectorcacheClient, VectorcacheError } from 'vectorcache';
import OpenAI from 'openai';

const vectorcache = new VectorcacheClient({
  apiKey: process.env.VECTORCACHE_API_KEY!,
  baseURL: 'https://api.vectorcache.com'
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY!
});

async function getChatResponse(prompt: string, context: string = ''): Promise<string> {
  try {
    // Try Vectorcache first
    const result = await vectorcache.query({
      prompt: prompt,
      context: context,
      model: 'gpt-4',
      similarity_threshold: 0.85
    });

    if (result.cache_hit) {
      console.log('✅ Cache hit! Saved time and money');
    } else {
      console.log('📝 Cache miss - new response generated');
    }

    return result.response;

  } catch (error: any) {
    return handleVectorcacheError(error, prompt, context);
  }
}

async function handleVectorcacheError(
  error: any,
  prompt: string,
  context: string
): Promise<string> {
  const status = error.response?.status;

  switch (status) {
    case 503:
      // Service unavailable - use fallback
      console.warn('⚠️ Vectorcache temporarily unavailable, calling LLM directly');
      const retryAfter = error.response?.headers['retry-after'] || 60;
      console.log(`   Retry after ${retryAfter} seconds`);
      return callLLMDirectly(prompt, context);

    case 422:
      // Content not cacheable
      console.warn('⚠️ Content not cacheable (may contain images/PDFs), calling LLM directly');
      return callLLMDirectly(prompt, context);

    case 429:
      // Rate limit exceeded
      console.error('❌ Monthly Vectorcache limit exceeded');
      const detail = error.response?.data;
      console.error(`   Usage: ${detail.current_usage}/${detail.monthly_limit}`);
      throw new Error('Vectorcache monthly limit reached. Please upgrade your plan.');

    case 401:
      // Invalid API key
      console.error('❌ Invalid Vectorcache API key');
      throw new Error('Vectorcache authentication failed. Check your API key.');

    case 500:
      // Internal server error - retry with exponential backoff
      console.error('❌ Vectorcache internal error, retrying...');
      await sleep(1000);
      try {
        return await getChatResponse(prompt, context);
      } catch {
        // Retry failed, fallback to direct LLM
        return callLLMDirectly(prompt, context);
      }

    default:
      // Unexpected error - fallback
      console.error('❌ Unexpected Vectorcache error:', error.message);
      return callLLMDirectly(prompt, context);
  }
}

async function callLLMDirectly(prompt: string, context: string = ''): Promise<string> {
  /**
   * Fallback: Call OpenAI directly when Vectorcache is unavailable
   */
  console.log('🔄 Falling back to direct OpenAI call');

  const messages: any[] = [];
  if (context) {
    messages.push({ role: 'system', content: context });
  }
  messages.push({ role: 'user', content: prompt });

  const response = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: messages
  });

  return response.choices[0].message.content || '';
}

function sleep(ms: number): Promise<void> {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// Usage
async function main() {
  const response = await getChatResponse(
    'What are the benefits of semantic caching?',
    'You are a helpful AI assistant'
  );
  console.log('Response:', response);
}

main();
```

## Python Error Handling

### Complete Example with Fallback

```python
import os
import time
from typing import Optional
from vectorcache import VectorcacheClient, VectorcacheError
from openai import OpenAI

vectorcache = VectorcacheClient(
    api_key=os.getenv("VECTORCACHE_API_KEY"),
    base_url="https://api.vectorcache.com"
)

openai_client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))


def get_chat_response(prompt: str, context: str = "") -> str:
    """
    Get chat response with Vectorcache, falling back to direct LLM on error.
    """
    try:
        # Try Vectorcache first
        result = vectorcache.query(
            prompt=prompt,
            context=context,
            model="gpt-4",
            similarity_threshold=0.85
        )

        if result['cache_hit']:
            print('✅ Cache hit! Saved time and money')
        else:
            print('📝 Cache miss - new response generated')

        return result['response']

    except VectorcacheError as e:
        return handle_vectorcache_error(e, prompt, context)


def handle_vectorcache_error(error: VectorcacheError, prompt: str, context: str = "") -> str:
    """
    Handle Vectorcache errors with appropriate fallback strategies.
    """
    status = error.status_code

    if status == 503:
        # Service unavailable - use fallback
        print('⚠️ Vectorcache temporarily unavailable, calling LLM directly')
        retry_after = error.retry_after or 60
        print(f'   Retry after {retry_after} seconds')
        return call_llm_directly(prompt, context)

    elif status == 422:
        # Content not cacheable
        print('⚠️ Content not cacheable (may contain images/PDFs), calling LLM directly')
        return call_llm_directly(prompt, context)

    elif status == 429:
        # Rate limit exceeded
        print(f'❌ Monthly Vectorcache limit exceeded: {error.message}')
        raise Exception('Vectorcache monthly limit reached. Please upgrade your plan.')

    elif status == 401:
        # Invalid API key
        print('❌ Invalid Vectorcache API key')
        raise Exception('Vectorcache authentication failed. Check your API key.')

    elif status == 500:
        # Internal server error - retry once with exponential backoff
        print('❌ Vectorcache internal error, retrying...')
        time.sleep(1)
        try:
            return get_chat_response(prompt, context)
        except:
            # Retry failed, fallback to direct LLM
            return call_llm_directly(prompt, context)

    else:
        # Unexpected error - fallback
        print(f'❌ Unexpected Vectorcache error: {error.message}')
        return call_llm_directly(prompt, context)


def call_llm_directly(prompt: str, context: str = "") -> str:
    """
    Fallback: Call OpenAI directly when Vectorcache is unavailable.
    """
    print('🔄 Falling back to direct OpenAI call')

    messages = []
    if context:
        messages.append({"role": "system", "content": context})
    messages.append({"role": "user", "content": prompt})

    response = openai_client.chat.completions.create(
        model="gpt-4",
        messages=messages
    )

    return response.choices[0].message.content


# Usage
if __name__ == "__main__":
    response = get_chat_response(
        prompt="What are the benefits of semantic caching?",
        context="You are a helpful AI assistant"
    )
    print(f"Response: {response}")
```

## Retry Strategies

### Exponential Backoff

For 500/503 errors, implement exponential backoff:

```typescript
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3,
  initialDelay: number = 1000
): Promise<T> {
  let lastError: any;

  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error: any) {
      lastError = error;

      // Don't retry on client errors (4xx)
      if (error.response?.status >= 400 && error.response?.status < 500) {
        throw error;
      }

      // Calculate delay: 1s, 2s, 4s, 8s...
      const delay = initialDelay * Math.pow(2, attempt);
      console.log(`Retry attempt ${attempt + 1}/${maxRetries} in ${delay}ms`);

      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }

  throw lastError;
}

// Usage
const result = await retryWithBackoff(() =>
  vectorcache.query({ prompt, context, model: 'gpt-4' })
);
```

## Respecting Retry-After Headers

When receiving 503 errors, always check the `Retry-After` header:

```typescript
if (error.response?.status === 503) {
  const retryAfter = parseInt(error.response.headers['retry-after'] || '60');
  console.log(`Service unavailable. Retry after ${retryAfter} seconds`);

  // Option 1: Wait and retry
  await new Promise(resolve => setTimeout(resolve, retryAfter * 1000));
  return await getChatResponse(prompt, context);

  // Option 2: Fallback immediately
  return callLLMDirectly(prompt, context);
}
```

## Timeout Configuration

Set appropriate timeouts to prevent hanging requests:

```typescript
// JavaScript/TypeScript
const vectorcache = new VectorcacheClient({
  apiKey: process.env.VECTORCACHE_API_KEY!,
  timeout: 10000  // 10 second timeout
});
```

```python
# Python
import httpx

vectorcache = VectorcacheClient(
    api_key=os.getenv("VECTORCACHE_API_KEY"),
    timeout=10.0  # 10 second timeout
)
```

## Health Check Monitoring

Monitor Vectorcache health before making requests:

```typescript
async function checkVectorcacheHealth(): Promise<boolean> {
  try {
    const response = await fetch('https://api.vectorcache.com/health/ready');
    const data = await response.json();

    if (response.status === 200 && data.status === 'ready') {
      return true;
    }

    console.warn('Vectorcache not ready:', data);
    return false;
  } catch (error) {
    console.error('Vectorcache health check failed:', error);
    return false;
  }
}

// Usage
const isHealthy = await checkVectorcacheHealth();
if (isHealthy) {
  // Use Vectorcache
  result = await vectorcache.query({...});
} else {
  // Skip cache, use LLM directly
  result = await callLLMDirectly(prompt);
}
```

## Production Checklist

Before deploying to production, ensure you:

- ✅ **Implement try-catch blocks** around all Vectorcache calls
- ✅ **Add fallback logic** to call your LLM directly on errors
- ✅ **Set appropriate timeouts** (recommended: 10 seconds)
- ✅ **Log errors** for monitoring and debugging
- ✅ **Respect Retry-After headers** for 503 errors
- ✅ **Implement exponential backoff** for retries
- ✅ **Handle 422 errors** (non-cacheable content) gracefully
- ✅ **Monitor rate limits** (429 errors) and alert users
- ✅ **Test error scenarios** in staging environment

## Common Patterns

### Pattern 1: Always Fallback

```typescript
async function getChatResponse(prompt: string): Promise<string> {
  try {
    const result = await vectorcache.query({...});
    return result.response;
  } catch {
    // Any error: fallback to LLM
    return await callLLMDirectly(prompt);
  }
}
```

### Pattern 2: Fail on Rate Limits

```typescript
async function getChatResponse(prompt: string): Promise<string> {
  try {
    const result = await vectorcache.query({...});
    return result.response;
  } catch (error: any) {
    if (error.response?.status === 429) {
      // Don't fallback on rate limits - force user to upgrade
      throw new Error('Monthly cache limit exceeded');
    }
    // Other errors: fallback
    return await callLLMDirectly(prompt);
  }
}
```

### Pattern 3: Circuit Breaker Client-Side

```typescript
class VectorcacheCircuitBreaker {
  private failureCount = 0;
  private lastFailureTime = 0;
  private readonly threshold = 5;
  private readonly timeout = 60000; // 60 seconds

  async call<T>(fn: () => Promise<T>): Promise<T> {
    // If too many recent failures, skip cache
    if (this.failureCount >= this.threshold) {
      if (Date.now() - this.lastFailureTime < this.timeout) {
        throw new Error('Circuit breaker open');
      }
      // Reset after timeout
      this.failureCount = 0;
    }

    try {
      const result = await fn();
      this.failureCount = 0; // Success resets counter
      return result;
    } catch (error) {
      this.failureCount++;
      this.lastFailureTime = Date.now();
      throw error;
    }
  }
}
```

## Next Steps

- [SDK Reference](/sdk/javascript/) - Detailed SDK documentation
- [API Reference](/api/query/) - Complete API specification
- [Best Practices](/guides/best-practices/) - General best practices
- [Security](/guides/security/) - Security considerations
