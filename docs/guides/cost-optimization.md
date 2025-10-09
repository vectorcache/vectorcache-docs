# Cost Optimization

Maximize your ROI with Vectorcache by optimizing cost savings.

## Understanding Costs

### LLM API Costs

Typical LLM API costs per query:

| Model | Cost per Query | Annual Cost (1000 queries/day) |
|-------|---------------|--------------------------------|
| GPT-4o | $0.002-0.005 | $730-1,825 |
| GPT-4o-mini | $0.0003-0.001 | $110-365 |
| Claude 3.5 Sonnet | $0.003-0.008 | $1,095-2,920 |
| Claude 3.5 Haiku | $0.0005-0.001 | $183-365 |

### Vectorcache Savings

With 50% cache hit rate:

| Model | Without Cache | With Vectorcache (50% hits) | Annual Savings |
|-------|---------------|----------------------------|----------------|
| GPT-4o | $1,825/year | $913/year | **$912/year** |
| Claude 3.5 Sonnet | $2,920/year | $1,460/year | **$1,460/year** |

*Assuming $29/month Vectorcache subscription*

## Calculating ROI

### Formula

```
ROI = (Cost Saved - Vectorcache Cost) / Vectorcache Cost × 100%

Cost Saved = (Cache Hits × Avg Query Cost)
```

### Example Calculation

```typescript
interface ROICalculation {
  monthlyQueries: number;
  cacheHitRate: number;
  avgQueryCost: number;
  vectorcacheCost: number;
}

function calculateROI(params: ROICalculation): number {
  const cacheHits = params.monthlyQueries * params.cacheHitRate;
  const costSaved = cacheHits * params.avgQueryCost;
  const netSavings = costSaved - params.vectorcacheCost;
  const roi = (netSavings / params.vectorcacheCost) * 100;

  return roi;
}

// Example: 30,000 queries/month, 50% hit rate, $0.003/query, $29/month
const roi = calculateROI({
  monthlyQueries: 30000,
  cacheHitRate: 0.50,
  avgQueryCost: 0.003,
  vectorcacheCost: 29
});

console.log(`ROI: ${roi.toFixed(0)}%`); // ~55% ROI
```

## Maximizing Cache Hit Rate

### 1. Optimize Similarity Threshold

Lower threshold = Higher hit rate (but less accuracy):

```typescript
// Test different thresholds
const thresholds = [0.80, 0.85, 0.90];

for (const threshold of thresholds) {
  const metrics = await testThreshold(threshold, testQueries);
  console.log(`Threshold ${threshold}: ${metrics.hitRate}% hits, $${metrics.saved}`);
}

// Choose threshold with best ROI
```

### 2. Use Context Segmentation

Segment cache by use case for better matches:

```typescript
// Separate caches for different contexts
await client.query({
  prompt: 'Reset password',
  context: 'customer-support-auth',
  model: 'gpt-4o'
});

await client.query({
  prompt: 'Reset password',
  context: 'admin-documentation',
  model: 'gpt-4o'
});
```

### 3. Normalize User Input

Preprocess queries for better matching:

```typescript
function normalizePrompt(prompt: string): string {
  return prompt
    .toLowerCase()
    .trim()
    .replace(/[^\w\s]/g, ' ')  // Remove punctuation
    .replace(/\s+/g, ' ');     // Normalize whitespace
}

const result = await client.query({
  prompt: normalizePrompt(userInput),
  model: 'gpt-4o'
});
```

## Cost Tracking

### Track Actual Savings

```typescript
class CostTracker {
  private totalSaved = 0;
  private totalQueries = 0;
  private cacheHits = 0;

  async query(request: CacheQueryRequest) {
    const result = await client.query(request);

    this.totalQueries++;
    if (result.cache_hit) {
      this.cacheHits++;
      this.totalSaved += result.cost_saved;
    }

    return result;
  }

  getMetrics() {
    const hitRate = (this.cacheHits / this.totalQueries * 100).toFixed(1);
    const avgSaved = this.totalSaved / this.cacheHits;

    return {
      totalQueries: this.totalQueries,
      cacheHits: this.cacheHits,
      hitRate: `${hitRate}%`,
      totalSaved: `$${this.totalSaved.toFixed(2)}`,
      avgSavedPerHit: `$${avgSaved.toFixed(4)}`
    };
  }
}
```

### Monthly Cost Analysis

```typescript
function analyzeMonthCosts(metrics: CacheMetrics) {
  const vectorcacheCost = 29; // Monthly subscription
  const costSaved = metrics.totalCostSaved;
  const netSavings = costSaved - vectorcacheCost;
  const roi = (netSavings / vectorcacheCost) * 100;

  return {
    vectorcacheCost: `$${vectorcacheCost}`,
    llmCostSaved: `$${costSaved.toFixed(2)}`,
    netSavings: `$${netSavings.toFixed(2)}`,
    roi: `${roi.toFixed(0)}%`,
    breakEven: netSavings >= 0
  };
}
```

## When Vectorcache Makes Sense

### ✅ Great Fit

- **High query volume**: 10,000+ queries/month
- **Repetitive queries**: Customer support, FAQs, documentation
- **Expensive models**: GPT-4o, Claude 3.5 Sonnet
- **Similar user questions**: Educational platforms, chatbots

### ⚠️ May Not Be Worth It

- **Low query volume**: <1,000 queries/month
- **Unique queries**: Each query is completely different
- **Cheap models only**: Using only GPT-4o-mini or similar
- **Real-time data**: Queries require latest information

### Break-Even Analysis

Minimum queries needed to break even (at different hit rates):

| Model | Cost/Query | 30% Hit Rate | 50% Hit Rate | 70% Hit Rate |
|-------|-----------|--------------|--------------|--------------|
| GPT-4o ($0.003) | $0.003 | ~32,000 | ~19,000 | ~14,000 |
| GPT-4o-mini ($0.0005) | $0.0005 | ~193,000 | ~116,000 | ~83,000 |
| Claude 3.5 Sonnet ($0.005) | $0.005 | ~19,000 | ~12,000 | ~8,000 |

*Monthly queries needed to break even at $29/month*

## Cost Optimization Strategies

### 1. Use Cheaper Models for Cache Misses

```typescript
async function smartQuery(prompt: string) {
  // Try cache first
  const result = await client.query({
    prompt,
    model: 'gpt-4o',
    similarityThreshold: 0.85
  });

  // If cache miss and query is simple, use cheaper model
  if (!result.cache_hit && isSimpleQuery(prompt)) {
    return await client.query({
      prompt,
      model: 'gpt-4o-mini', // Cheaper alternative
      similarityThreshold: 0.85
    });
  }

  return result;
}
```

### 2. Batch Similar Queries

Group related queries to maximize cache hits:

```typescript
async function batchQuery(prompts: string[]) {
  // Group similar prompts
  const groups = groupSimilarPrompts(prompts);

  // Query each group once, reuse for similar prompts
  const results = [];
  for (const group of groups) {
    const result = await client.query({
      prompt: group[0], // Use first as representative
      model: 'gpt-4o'
    });

    // Reuse result for all similar prompts
    group.forEach(prompt => {
      results.push({ prompt, result });
    });
  }

  return results;
}
```

### 3. Implement Smart Caching Logic

```typescript
async function intelligentCache(prompt: string, userContext: any) {
  // Don't cache unique/time-sensitive queries
  if (isTimeSensitive(prompt) || isUserSpecific(prompt, userContext)) {
    return await directLLMCall(prompt);
  }

  // Use cache for general queries
  return await client.query({
    prompt,
    model: 'gpt-4o',
    similarityThreshold: 0.85
  });
}

function isTimeSensitive(prompt: string): boolean {
  const timeKeywords = ['today', 'now', 'current', 'latest', 'recent'];
  return timeKeywords.some(kw => prompt.toLowerCase().includes(kw));
}
```

## Monitoring ROI

### Dashboard Metrics

Track these metrics in your dashboard:

```typescript
interface ROIDashboard {
  period: string;
  totalQueries: number;
  cacheHits: number;
  hitRate: string;
  llmCostSaved: string;
  vectorcacheCost: string;
  netSavings: string;
  roi: string;
}

function generateROIDashboard(metrics: CacheMetrics): ROIDashboard {
  const vectorcacheCost = 29;
  const netSavings = metrics.totalCostSaved - vectorcacheCost;
  const roi = (netSavings / vectorcacheCost) * 100;

  return {
    period: 'This Month',
    totalQueries: metrics.totalQueries,
    cacheHits: metrics.cacheHits,
    hitRate: `${(metrics.cacheHits / metrics.totalQueries * 100).toFixed(1)}%`,
    llmCostSaved: `$${metrics.totalCostSaved.toFixed(2)}`,
    vectorcacheCost: `$${vectorcacheCost}`,
    netSavings: `$${netSavings.toFixed(2)}`,
    roi: `${roi.toFixed(0)}%`
  };
}
```

### Alerts for Poor ROI

```typescript
function checkROI(metrics: CacheMetrics) {
  const vectorcacheCost = 29;
  const netSavings = metrics.totalCostSaved - vectorcacheCost;

  if (netSavings < 0 && metrics.totalQueries > 1000) {
    alert('Negative ROI detected', {
      netSavings: `$${netSavings.toFixed(2)}`,
      suggestion: 'Consider lowering similarity threshold or increasing query volume'
    });
  }
}
```

## Real-World Examples

### Example 1: Customer Support Chatbot

**Scenario:**
- 50,000 queries/month
- 60% cache hit rate
- GPT-4o ($0.003/query)
- $29/month Vectorcache

**Calculation:**
```
Cache hits: 50,000 × 0.60 = 30,000
Cost saved: 30,000 × $0.003 = $90
Net savings: $90 - $29 = $61/month
ROI: ($61 / $29) × 100 = 210%
Annual savings: $732
```

### Example 2: Educational Platform

**Scenario:**
- 100,000 queries/month
- 70% cache hit rate (high due to repetitive educational questions)
- GPT-4o ($0.003/query)
- $29/month Vectorcache

**Calculation:**
```
Cache hits: 100,000 × 0.70 = 70,000
Cost saved: 70,000 × $0.003 = $210
Net savings: $210 - $29 = $181/month
ROI: ($181 / $29) × 100 = 624%
Annual savings: $2,172
```

## Best Practices

1. **Track metrics religiously** - Know your exact hit rate and cost savings
2. **Test thresholds** - Find the sweet spot for your use case
3. **Segment caches** - Use context for better organization
4. **Monitor ROI** - Alert when ROI drops below acceptable level
5. **Optimize continuously** - Adjust based on real data

## Next Steps

- [Best Practices](best-practices.md) - Production tips
- [Similarity Tuning](similarity-tuning.md) - Optimize hit rate
- [API Reference](../api/overview.md) - API documentation
