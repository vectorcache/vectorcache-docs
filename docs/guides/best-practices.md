# Best Practices

Production-ready tips for deploying Vectorcache in your application.

## Security

### 1. Protect API Keys

❌ **Never** hardcode API keys:

```javascript
// DON'T DO THIS
const apiKey = 'vc_1234567890abcdef';
```

✅ Use environment variables:

```javascript
// DO THIS
const apiKey = process.env.VECTORCACHE_API_KEY;
```

### 2. Separate Keys per Environment

Use different API keys for development, staging, and production:

```bash
# .env.development
VECTORCACHE_API_KEY=vc_dev_key...

# .env.staging
VECTORCACHE_API_KEY=vc_staging_key...

# .env.production
VECTORCACHE_API_KEY=vc_prod_key...
```

### 3. Rotate Keys Regularly

- Create new keys every 90 days
- Revoke old keys after migration
- Monitor key usage in dashboard

### 4. Never Commit Keys to Git

Add to `.gitignore`:

```bash
.env
.env.local
.env.*.local
config/secrets.json
```

## Error Handling

### 1. Always Handle Both Cache Hit and Miss

```typescript
const result = await client.query({ prompt, model });

if (result.cache_hit) {
  console.log(`Cache hit! Saved $${result.cost_saved}`);
} else {
  console.log('Cache miss - stored for future use');
}

// Your app should work regardless of cache_hit value
return result.response;
```

### 2. Implement Retry Logic

```typescript
async function queryWithRetry(request, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await client.query(request);
    } catch (error) {
      if (error.statusCode === 429 && attempt < maxRetries) {
        const delay = Math.min(1000 * Math.pow(2, attempt), 10000);
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
}
```

### 3. Graceful Degradation

Have a fallback when Vectorcache is unavailable:

```typescript
async function getResponse(prompt: string) {
  try {
    const result = await client.query({ prompt, model: 'gpt-4o' });
    return result.response;
  } catch (error) {
    console.error('Vectorcache error, using fallback:', error);
    return await directLLMCall(prompt);
  }
}
```

## Performance Optimization

### 1. Set Appropriate Timeouts

```typescript
const client = new VectorcacheClient({
  apiKey: process.env.VECTORCACHE_API_KEY!,
  timeout: 10000 // 10 seconds for production
});
```

### 2. Use Connection Pooling

For high-throughput applications:

```python
# Python example with connection pooling
import requests
from requests.adapters import HTTPAdapter
from requests.packages.urllib3.util.retry import Retry

session = requests.Session()
retry = Retry(total=3, backoff_factor=0.3)
adapter = HTTPAdapter(max_retries=retry, pool_maxsize=100)
session.mount('https://', adapter)
```

### 3. Batch Similar Requests

Group related queries together:

```typescript
const prompts = [
  'What is ML?',
  'Explain deep learning',
  'What are neural networks?'
];

const results = await Promise.all(
  prompts.map(prompt => client.query({ prompt, model: 'gpt-4o' }))
);
```

### 4. Monitor Cache Performance

Track metrics to optimize threshold:

```typescript
let totalQueries = 0;
let cacheHits = 0;

async function trackQuery(prompt: string) {
  const result = await client.query({ prompt, model: 'gpt-4o' });

  totalQueries++;
  if (result.cache_hit) cacheHits++;

  const hitRate = (cacheHits / totalQueries * 100).toFixed(1);
  console.log(`Cache hit rate: ${hitRate}%`);

  return result;
}
```

## Caching Strategy

### 1. Use Context for Segmentation

Separate caches by use case:

```typescript
// Customer support queries
await client.query({
  prompt: 'How do I reset my password?',
  context: 'customer-support',
  model: 'gpt-4o'
});

// Internal documentation
await client.query({
  prompt: 'How do I reset my password?',
  context: 'internal-docs',
  model: 'gpt-4o'
});
```

### 2. Choose Similarity Thresholds Wisely

| Use Case | Recommended Threshold |
|----------|----------------------|
| Legal/Medical | 0.92-0.95 |
| Customer Support | 0.85-0.90 |
| Education/FAQs | 0.80-0.85 |
| General Content | 0.85 |

### 3. Test Thresholds with Real Data

```typescript
const thresholds = [0.80, 0.85, 0.90, 0.95];

for (const threshold of thresholds) {
  const result = await client.query({
    prompt: testPrompt,
    model: 'gpt-4o',
    similarityThreshold: threshold
  });

  console.log(`Threshold ${threshold}: ${result.cache_hit ? 'HIT' : 'MISS'}`);
  if (result.similarity_score) {
    console.log(`Score: ${result.similarity_score}`);
  }
}
```

## Monitoring

### 1. Track Key Metrics

Monitor these metrics in your application:

```typescript
interface CacheMetrics {
  totalQueries: number;
  cacheHits: number;
  cacheMisses: number;
  totalCostSaved: number;
  averageSimilarityScore: number;
  errorCount: number;
}

const metrics: CacheMetrics = {
  totalQueries: 0,
  cacheHits: 0,
  cacheMisses: 0,
  totalCostSaved: 0,
  averageSimilarityScore: 0,
  errorCount: 0
};

async function queryAndTrack(request) {
  try {
    const result = await client.query(request);

    metrics.totalQueries++;
    if (result.cache_hit) {
      metrics.cacheHits++;
      metrics.totalCostSaved += result.cost_saved;
    } else {
      metrics.cacheMisses++;
    }

    return result;
  } catch (error) {
    metrics.errorCount++;
    throw error;
  }
}
```

### 2. Log Important Events

```typescript
function logCacheEvent(result: CacheQueryResponse, request: CacheQueryRequest) {
  const logEntry = {
    timestamp: new Date().toISOString(),
    cache_hit: result.cache_hit,
    similarity_score: result.similarity_score,
    cost_saved: result.cost_saved,
    model: request.model,
    context: request.context,
    threshold: request.similarityThreshold
  };

  logger.info('cache_query', logEntry);

  // Alert on low hit rates
  const hitRate = metrics.cacheHits / metrics.totalQueries;
  if (hitRate < 0.3 && metrics.totalQueries > 100) {
    logger.warn('Low cache hit rate', { hitRate, metrics });
  }
}
```

### 3. Set Up Alerts

Alert on important conditions:

```typescript
function checkAndAlert(metrics: CacheMetrics) {
  // High error rate
  const errorRate = metrics.errorCount / metrics.totalQueries;
  if (errorRate > 0.05) {
    alert('High Vectorcache error rate', { errorRate, metrics });
  }

  // Low hit rate
  const hitRate = metrics.cacheHits / metrics.totalQueries;
  if (hitRate < 0.3 && metrics.totalQueries > 100) {
    alert('Low cache hit rate', { hitRate, metrics });
  }

  // High cost (check if caching is actually saving money)
  const expectedSavings = metrics.totalQueries * 0.003; // Assume $0.003 per query
  if (metrics.totalCostSaved < expectedSavings * 0.5) {
    alert('Lower than expected cost savings', { metrics });
  }
}
```

## Testing

### 1. Unit Tests

Test your Vectorcache integration:

```typescript
import { VectorcacheClient } from 'vectorcache-js';

describe('Vectorcache Integration', () => {
  const client = new VectorcacheClient({
    apiKey: process.env.TEST_VECTORCACHE_KEY!
  });

  it('should handle cache hits', async () => {
    const prompt = 'Test prompt for caching';

    // First call - cache miss
    const result1 = await client.query({ prompt, model: 'gpt-4o' });
    expect(result1.cache_hit).toBe(false);

    // Second call - cache hit
    const result2 = await client.query({ prompt, model: 'gpt-4o' });
    expect(result2.cache_hit).toBe(true);
    expect(result2.similarity_score).toBeGreaterThan(0.9);
  });

  it('should handle errors gracefully', async () => {
    const invalidClient = new VectorcacheClient({ apiKey: 'invalid' });

    await expect(
      invalidClient.query({ prompt: 'test', model: 'gpt-4o' })
    ).rejects.toThrow('Invalid API key');
  });
});
```

### 2. Load Testing

Test with realistic load:

```typescript
async function loadTest(concurrency: number, totalRequests: number) {
  const results = [];
  const batchSize = concurrency;

  for (let i = 0; i < totalRequests; i += batchSize) {
    const batch = Array(Math.min(batchSize, totalRequests - i))
      .fill(null)
      .map(() => client.query({
        prompt: `Test query ${i}`,
        model: 'gpt-4o'
      }));

    const batchResults = await Promise.all(batch);
    results.push(...batchResults);
  }

  const hitRate = results.filter(r => r.cache_hit).length / results.length;
  console.log(`Load test complete: ${hitRate * 100}% hit rate`);
}
```

## Cost Optimization

### 1. Track ROI

Monitor actual cost savings:

```typescript
interface CostAnalysis {
  totalQueries: number;
  cacheHits: number;
  totalCostSaved: number;
  vectorcacheCost: number; // Your Vectorcache subscription
  netSavings: number;
}

function analyzeCosts(metrics: CacheMetrics): CostAnalysis {
  const vectorcacheCost = 29; // Example: $29/month plan

  return {
    totalQueries: metrics.totalQueries,
    cacheHits: metrics.cacheHits,
    totalCostSaved: metrics.totalCostSaved,
    vectorcacheCost,
    netSavings: metrics.totalCostSaved - vectorcacheCost
  };
}
```

### 2. Optimize Threshold Based on ROI

```typescript
function findOptimalThreshold(testPrompts: string[]) {
  const thresholds = [0.75, 0.80, 0.85, 0.90, 0.95];
  const results = [];

  for (const threshold of thresholds) {
    let hits = 0;
    let totalSaved = 0;

    for (const prompt of testPrompts) {
      const result = await client.query({
        prompt,
        model: 'gpt-4o',
        similarityThreshold: threshold
      });

      if (result.cache_hit) {
        hits++;
        totalSaved += result.cost_saved;
      }
    }

    results.push({
      threshold,
      hitRate: hits / testPrompts.length,
      totalSaved
    });
  }

  // Find threshold with best ROI
  return results.sort((a, b) => b.totalSaved - a.totalSaved)[0];
}
```

## Production Checklist

Before going live:

- [ ] API keys stored in environment variables
- [ ] Separate keys for dev/staging/production
- [ ] Error handling implemented with retries
- [ ] Fallback logic for service unavailability
- [ ] Timeouts configured appropriately
- [ ] Similarity threshold tested with real data
- [ ] Context used for cache segmentation
- [ ] Metrics and logging implemented
- [ ] Alerts configured for errors and low hit rates
- [ ] Load testing completed
- [ ] Cost analysis reviewed
- [ ] `.env` files in `.gitignore`

## Next Steps

- [Similarity Tuning Guide](similarity-tuning.md) - Optimize cache performance
- [Cost Optimization](cost-optimization.md) - Maximize savings
- [Security Guide](security.md) - Advanced security practices
