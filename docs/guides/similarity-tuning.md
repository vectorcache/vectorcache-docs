# Similarity Threshold Tuning

Optimize your cache hit rate by tuning the similarity threshold parameter.

## Understanding Similarity Threshold

The `similarity_threshold` parameter determines how similar two prompts must be for a cache hit. It ranges from 0 to 1:

- **1.0** = Exact match only
- **0.95-0.99** = Nearly identical
- **0.85-0.94** = Very similar (recommended)
- **0.75-0.84** = Somewhat similar
- **0.0-0.74** = Loosely similar (not recommended)

## How Similarity Works

Vectorcache uses cosine similarity between vector embeddings:

```
similarity_score = cosine_similarity(embedding_A, embedding_B)

if similarity_score >= similarity_threshold:
    return cached_response  # Cache hit
else:
    call_llm()              # Cache miss
```

### Example Similarity Scores

Real examples from production:

| Prompt 1 | Prompt 2 | Score | Match at 0.85? |
|----------|----------|-------|----------------|
| "What is ML?" | "What is machine learning?" | 0.94 | ✅ Yes |
| "Explain AI" | "What is artificial intelligence?" | 0.88 | ✅ Yes |
| "Python tutorial" | "How to learn Python" | 0.82 | ❌ No |
| "Reset password" | "Change password" | 0.79 | ❌ No |
| "Order status" | "Track my order" | 0.91 | ✅ Yes |

## Finding Your Optimal Threshold

### Step 1: Start with Default (0.85)

Begin with the recommended default:

```typescript
const result = await client.query({
  prompt: userQuery,
  model: 'gpt-4o',
  similarityThreshold: 0.85  // Default
});
```

### Step 2: Monitor Similarity Scores

Track actual similarity scores from cache hits:

```typescript
const scores: number[] = [];

async function trackQuery(prompt: string) {
  const result = await client.query({
    prompt,
    model: 'gpt-4o',
    similarityThreshold: 0.85
  });

  if (result.cache_hit && result.similarity_score) {
    scores.push(result.similarity_score);
  }

  return result;
}

// Analyze scores after 100+ queries
function analyzeScores() {
  const avg = scores.reduce((a, b) => a + b, 0) / scores.length;
  const min = Math.min(...scores);
  const max = Math.max(...scores);

  console.log(`Average: ${avg.toFixed(3)}`);
  console.log(`Min: ${min.toFixed(3)}`);
  console.log(`Max: ${max.toFixed(3)}`);
}
```

### Step 3: Test Different Thresholds

Run A/B tests with various thresholds:

```typescript
async function testThresholds(prompts: string[]) {
  const thresholds = [0.75, 0.80, 0.85, 0.90, 0.95];

  for (const threshold of thresholds) {
    let hits = 0;
    const scores: number[] = [];

    for (const prompt of prompts) {
      const result = await client.query({
        prompt,
        model: 'gpt-4o',
        similarityThreshold: threshold
      });

      if (result.cache_hit) {
        hits++;
        if (result.similarity_score) {
          scores.push(result.similarity_score);
        }
      }
    }

    const hitRate = (hits / prompts.length * 100).toFixed(1);
    const avgScore = scores.length > 0
      ? (scores.reduce((a, b) => a + b, 0) / scores.length).toFixed(3)
      : 'N/A';

    console.log(`Threshold ${threshold}: ${hitRate}% hits, avg score: ${avgScore}`);
  }
}
```

### Step 4: Choose Based on Use Case

Select threshold based on your requirements:

| Use Case | Priority | Recommended Threshold |
|----------|----------|----------------------|
| Legal/Medical | Accuracy | 0.92-0.95 |
| Financial | Accuracy | 0.90-0.93 |
| Customer Support | Balance | 0.85-0.90 |
| Educational | Hit Rate | 0.80-0.85 |
| FAQs | Hit Rate | 0.80-0.85 |
| General Content | Balance | 0.85 |

## Use Case Examples

### High Precision: Legal Advice

For legal content, false positives are costly:

```typescript
const legalQuery = await client.query({
  prompt: 'Interpret clause 5.2 of the agreement',
  context: 'legal-contract-review',
  model: 'gpt-4o',
  similarityThreshold: 0.93  // High threshold for accuracy
});
```

**Why 0.93?**
- Legal queries must be very specific
- Different clauses require different interpretations
- Cost of wrong answer > cost of LLM call

### Balanced: Customer Support

For support chatbots, balance hit rate and accuracy:

```typescript
const supportQuery = await client.query({
  prompt: 'How do I reset my password?',
  context: 'customer-support',
  model: 'gpt-4o',
  similarityThreshold: 0.87  // Balanced threshold
});
```

**Why 0.87?**
- Similar questions should get same answer
- "Reset password" vs "Change password" should match
- Most support queries have common variations

### High Hit Rate: Educational FAQs

For educational content, maximize cache hits:

```typescript
const eduQuery = await client.query({
  prompt: 'What is photosynthesis?',
  context: 'biology-education',
  model: 'gpt-4o',
  similarityThreshold: 0.82  // Lower threshold for more hits
});
```

**Why 0.82?**
- Educational questions have many phrasings
- "What is X?" vs "Explain X" vs "Define X" should match
- General explanations are reusable

## Dynamic Threshold Adjustment

Adjust threshold based on context:

```typescript
function getThreshold(context: string): number {
  const thresholdMap: Record<string, number> = {
    'legal': 0.93,
    'medical': 0.92,
    'financial': 0.90,
    'support': 0.87,
    'education': 0.82,
    'default': 0.85
  };

  return thresholdMap[context] || thresholdMap['default'];
}

// Usage
const result = await client.query({
  prompt: userQuery,
  context: userContext,
  model: 'gpt-4o',
  similarityThreshold: getThreshold(userContext)
});
```

## Measuring Impact

### Hit Rate vs Threshold

Track how threshold affects hit rate:

```typescript
interface ThresholdMetrics {
  threshold: number;
  queries: number;
  hits: number;
  hitRate: number;
  avgSimilarity: number;
  costSaved: number;
}

async function measureImpact(
  prompts: string[],
  threshold: number
): Promise<ThresholdMetrics> {
  let hits = 0;
  let totalSimilarity = 0;
  let costSaved = 0;

  for (const prompt of prompts) {
    const result = await client.query({
      prompt,
      model: 'gpt-4o',
      similarityThreshold: threshold
    });

    if (result.cache_hit) {
      hits++;
      totalSimilarity += result.similarity_score || 0;
      costSaved += result.cost_saved;
    }
  }

  return {
    threshold,
    queries: prompts.length,
    hits,
    hitRate: hits / prompts.length,
    avgSimilarity: hits > 0 ? totalSimilarity / hits : 0,
    costSaved
  };
}
```

### Quality vs Quantity Trade-off

Higher threshold = Higher quality, Lower hit rate:

```
Threshold 0.95: 15% hit rate, $45/month saved   (high quality)
Threshold 0.90: 35% hit rate, $105/month saved  (balanced)
Threshold 0.85: 50% hit rate, $150/month saved  (balanced)
Threshold 0.80: 65% hit rate, $195/month saved  (quantity)
Threshold 0.75: 75% hit rate, $225/month saved  (risky)
```

**Recommendation**: Choose the highest threshold that still gives you acceptable hit rate.

## Advanced Techniques

### Context-Specific Thresholds

Use different thresholds for different contexts:

```typescript
const contextThresholds = {
  'legal-review': 0.93,
  'medical-advice': 0.92,
  'customer-support': 0.87,
  'product-info': 0.85,
  'general-faq': 0.82
};

async function queryWithContextThreshold(
  prompt: string,
  context: string
) {
  const threshold = contextThresholds[context] || 0.85;

  return await client.query({
    prompt,
    context,
    model: 'gpt-4o',
    similarityThreshold: threshold
  });
}
```

### Adaptive Thresholds

Adjust threshold based on user feedback:

```typescript
class AdaptiveCache {
  private threshold = 0.85;
  private feedbackScores: number[] = [];

  async query(prompt: string) {
    return await client.query({
      prompt,
      model: 'gpt-4o',
      similarityThreshold: this.threshold
    });
  }

  recordFeedback(helpful: boolean, similarityScore?: number) {
    if (!similarityScore) return;

    // If user found it helpful, this score is good
    if (helpful) {
      this.feedbackScores.push(similarityScore);
    }

    // Adjust threshold based on feedback
    if (this.feedbackScores.length >= 20) {
      const avgGoodScore = this.feedbackScores.reduce((a, b) => a + b) /
                          this.feedbackScores.length;

      // Set threshold slightly below average good score
      this.threshold = Math.max(0.75, avgGoodScore - 0.05);

      console.log(`Adjusted threshold to ${this.threshold.toFixed(2)}`);
      this.feedbackScores = []; // Reset for next period
    }
  }
}
```

### Multi-Tier Thresholds

Try high threshold first, fall back to lower:

```typescript
async function multiTierQuery(prompt: string) {
  // Try high precision first
  let result = await client.query({
    prompt,
    model: 'gpt-4o',
    similarityThreshold: 0.92
  });

  if (result.cache_hit) {
    return { ...result, tier: 'high-precision' };
  }

  // Fall back to balanced
  result = await client.query({
    prompt,
    model: 'gpt-4o',
    similarityThreshold: 0.85
  });

  return { ...result, tier: result.cache_hit ? 'balanced' : 'miss' };
}
```

## Common Pitfalls

### ❌ Setting Threshold Too Low

```typescript
// DON'T: Too many false positives
similarityThreshold: 0.65  // Will match unrelated queries
```

**Problem**: You'll get irrelevant cached responses

### ❌ Setting Threshold Too High

```typescript
// DON'T: Rarely any cache hits
similarityThreshold: 0.98  // Only exact matches
```

**Problem**: Very low hit rate, wasting cache potential

### ❌ Not Testing with Real Data

```typescript
// DON'T: Guess based on assumptions
similarityThreshold: 0.85  // "I think this will work"
```

**Solution**: Always test with actual user queries

### ✅ The Right Approach

```typescript
// DO: Test and measure
const optimalThreshold = await findOptimalThreshold(realUserQueries);
```

## Monitoring and Alerts

Set up alerts for threshold issues:

```typescript
function monitorThreshold(metrics: CacheMetrics) {
  const hitRate = metrics.cacheHits / metrics.totalQueries;
  const avgSimilarity = metrics.avgSimilarityScore;

  // Hit rate too low
  if (hitRate < 0.2 && metrics.totalQueries > 100) {
    alert('Low cache hit rate - consider lowering threshold', {
      hitRate,
      currentThreshold: 0.85
    });
  }

  // Similarity scores too close to threshold
  if (avgSimilarity < 0.88 && currentThreshold === 0.85) {
    alert('Avg similarity close to threshold - may need adjustment', {
      avgSimilarity,
      currentThreshold: 0.85
    });
  }
}
```

## Summary

**Quick Reference:**

| Scenario | Recommended Threshold |
|----------|----------------------|
| Just starting | 0.85 |
| Need high accuracy | 0.90-0.95 |
| Want high hit rate | 0.80-0.85 |
| Legal/Medical | 0.92-0.95 |
| Customer support | 0.85-0.90 |
| Education/FAQs | 0.80-0.85 |

**Process:**
1. Start with 0.85
2. Monitor actual similarity scores
3. Test different thresholds with real data
4. Choose based on your quality vs quantity needs
5. Adjust based on user feedback

## Next Steps

- [Best Practices](best-practices.md) - Production deployment tips
- [Cost Optimization](cost-optimization.md) - Maximize ROI
- [API Reference](../api/cache.md) - Complete API docs
