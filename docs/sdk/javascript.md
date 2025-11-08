# JavaScript/TypeScript SDK

Complete guide to the Vectorcache JavaScript/TypeScript SDK.

## Installation

```bash
npm install vectorcache
```

## Quick Start

```typescript
import { VectorcacheClient } from 'vectorcache';

const client = new VectorcacheClient({
  apiKey: process.env.VECTORCACHE_API_KEY!,
  projectId: process.env.VECTORCACHE_PROJECT_ID!,
});

const result = await client.query({
  prompt: 'What is machine learning?',
  model: 'gpt-4o',
  similarityThreshold: 0.85
});

console.log(result.response);
```

## API Reference

### VectorcacheClient

#### Constructor

```typescript
new VectorcacheClient(config: VectorcacheConfig)
```

**Parameters:**

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `config.apiKey` | string | Yes | - | Your Vectorcache API key |
| `config.projectId` | string | Yes | - | Your Vectorcache project ID |
| `config.baseUrl` | string | No | `https://api.vectorcache.ai` | API base URL |
| `config.timeout` | number | No | `30000` | Request timeout (ms) |

#### Methods

##### `query()`

Query the semantic cache.

```typescript
async query(request: CacheQueryRequest): Promise<CacheQueryResponse>
```

**Request Parameters:**

```typescript
interface CacheQueryRequest {
  prompt: string;                    // Required: The prompt to cache/query
  model: string;                     // Required: LLM model (e.g., 'gpt-4o')
  context?: string;                  // Optional: Additional context
  similarityThreshold?: number;      // Optional: 0-1, default 0.85
  includeDebug?: boolean;           // Optional: Include debug info
}
```

**Response:**

```typescript
interface CacheQueryResponse {
  cache_hit: boolean;               // Whether the query hit the cache
  response: string;                 // The LLM response
  similarity_score: number | null;  // Similarity score (null on cache miss)
  cost_saved: number;              // Cost saved in USD (0 on cache miss)
  llm_provider: string;            // Provider used ('cache' or LLM name)
  debug?: DebugInfo;               // Debug information (if requested)
}
```

**Example:**

```typescript
const result = await client.query({
  prompt: 'Explain quantum computing',
  context: 'Educational content',
  model: 'gpt-4o',
  similarityThreshold: 0.85,
  includeDebug: true
});

if (result.cache_hit) {
  console.log(`Cache hit! Similarity: ${result.similarity_score}`);
  console.log(`Cost saved: $${result.cost_saved}`);
} else {
  console.log('Cache miss - fetched from LLM');
}
```

## TypeScript Support

The SDK is written in TypeScript and includes full type definitions.

### Import Types

```typescript
import type {
  VectorcacheClient,
  VectorcacheConfig,
  CacheQueryRequest,
  CacheQueryResponse,
  VectorcacheError
} from 'vectorcache';
```

### Type-Safe Queries

```typescript
const request: CacheQueryRequest = {
  prompt: 'What is AI?',
  model: 'gpt-4o',
  similarityThreshold: 0.85
};

const response: CacheQueryResponse = await client.query(request);
```

## Error Handling

### Error Types

The SDK throws typed errors:

```typescript
class VectorcacheError extends Error {
  statusCode?: number;
  response?: any;
}
```

### Handling Errors

```typescript
import { VectorcacheError } from 'vectorcache';

try {
  const result = await client.query({
    prompt: 'What is AI?',
    model: 'gpt-4o'
  });
} catch (error) {
  if (error instanceof VectorcacheError) {
    switch (error.statusCode) {
      case 401:
        console.error('Authentication failed - check your API key');
        break;
      case 429:
        console.error('Rate limit exceeded');
        break;
      case 500:
        console.error('Server error:', error.message);
        break;
      default:
        console.error('Unexpected error:', error);
    }
  } else {
    console.error('Network or unknown error:', error);
  }
}
```

### Retry Logic

Implement retry logic for transient failures:

```typescript
async function queryWithRetry(
  client: VectorcacheClient,
  request: CacheQueryRequest,
  maxRetries = 3
): Promise<CacheQueryResponse> {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await client.query(request);
    } catch (error) {
      if (error instanceof VectorcacheError && error.statusCode === 429) {
        // Rate limit - wait before retry
        const delay = Math.min(1000 * Math.pow(2, attempt), 10000);
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error; // Don't retry other errors
    }
  }
  throw new Error('Max retries exceeded');
}
```

## Advanced Usage

### Context-Aware Caching

Use context to segment your cache:

```typescript
// Educational queries
const eduResult = await client.query({
  prompt: 'What is photosynthesis?',
  context: 'biology-education',
  model: 'gpt-4o'
});

// Technical documentation
const docResult = await client.query({
  prompt: 'What is photosynthesis?',
  context: 'technical-documentation',
  model: 'gpt-4o'
});
```

These will be cached separately even though the prompt is the same.

### Multiple Clients

Create separate clients for different projects:

```typescript
const productionClient = new VectorcacheClient({
  apiKey: process.env.PROD_VECTORCACHE_KEY!,
});

const stagingClient = new VectorcacheClient({
  apiKey: process.env.STAGING_VECTORCACHE_KEY!,
  baseUrl: 'https://staging-api.vectorcache.ai'
});
```

### Streaming Responses (Coming Soon)

```typescript
// Future feature
const stream = await client.queryStream({
  prompt: 'Long explanation...',
  model: 'gpt-4o'
});

for await (const chunk of stream) {
  process.stdout.write(chunk);
}
```

## Framework Integration

### Next.js

```typescript
// app/api/chat/route.ts
import { VectorcacheClient } from 'vectorcache';
import { NextRequest, NextResponse } from 'next/server';

const client = new VectorcacheClient({
  apiKey: process.env.VECTORCACHE_API_KEY!,
});

export async function POST(request: NextRequest) {
  const { prompt } = await request.json();

  const result = await client.query({
    prompt,
    model: 'gpt-4o',
    similarityThreshold: 0.85
  });

  return NextResponse.json(result);
}
```

### Express

```typescript
import express from 'express';
import { VectorcacheClient } from 'vectorcache';

const app = express();
const client = new VectorcacheClient({
  apiKey: process.env.VECTORCACHE_API_KEY!,
});

app.post('/api/query', async (req, res) => {
  try {
    const result = await client.query({
      prompt: req.body.prompt,
      model: 'gpt-4o'
    });
    res.json(result);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

### React Hook

```typescript
import { useState } from 'react';
import { VectorcacheClient } from 'vectorcache';

const client = new VectorcacheClient({
  apiKey: process.env.NEXT_PUBLIC_VECTORCACHE_KEY!,
});

export function useVectorcache() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  const query = async (prompt: string, model: string) => {
    setLoading(true);
    setError(null);
    try {
      const result = await client.query({ prompt, model });
      return result;
    } catch (err) {
      setError(err as Error);
      throw err;
    } finally {
      setLoading(false);
    }
  };

  return { query, loading, error };
}
```

## Examples

### Chatbot with Cache Metrics

```typescript
import { VectorcacheClient } from 'vectorcache';

const client = new VectorcacheClient({
  apiKey: process.env.VECTORCACHE_API_KEY!,
});

let totalQueries = 0;
let cacheHits = 0;
let totalCostSaved = 0;

async function chat(userMessage: string): Promise<string> {
  const result = await client.query({
    prompt: userMessage,
    context: 'customer-support-bot',
    model: 'gpt-4o',
    similarityThreshold: 0.85
  });

  totalQueries++;
  if (result.cache_hit) {
    cacheHits++;
    totalCostSaved += result.cost_saved;
  }

  console.log(`Cache hit rate: ${(cacheHits / totalQueries * 100).toFixed(1)}%`);
  console.log(`Total cost saved: $${totalCostSaved.toFixed(4)}`);

  return result.response;
}
```

### Batch Processing

```typescript
async function processBatch(prompts: string[]) {
  const results = await Promise.all(
    prompts.map(prompt =>
      client.query({
        prompt,
        model: 'gpt-4o',
        similarityThreshold: 0.85
      })
    )
  );

  const hitRate = results.filter(r => r.cache_hit).length / results.length;
  const totalSaved = results.reduce((sum, r) => sum + r.cost_saved, 0);

  console.log(`Batch hit rate: ${(hitRate * 100).toFixed(1)}%`);
  console.log(`Batch cost saved: $${totalSaved.toFixed(4)}`);

  return results;
}
```

## Best Practices

1. **Store API keys securely** - Use environment variables
2. **Handle errors gracefully** - Implement proper error handling
3. **Set appropriate timeouts** - Based on your use case
4. **Monitor cache performance** - Track hit rates and cost savings
5. **Use context wisely** - Segment caches by use case
6. **Test similarity thresholds** - Find optimal values for your data

## Next Steps

- [Configuration Guide](../getting-started/configuration.md)
- [API Reference](../api/overview.md)
- [Best Practices](../guides/best-practices.md)
