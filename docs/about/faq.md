# Frequently Asked Questions

Common questions about Vectorcache.

## General

### What is Vectorcache?

Vectorcache is a semantic caching layer for LLM applications. Unlike traditional caching that only matches exact queries, Vectorcache uses vector embeddings to match semantically similar queries, dramatically improving cache hit rates.

### How does semantic caching work?

When you send a query:
1. Vectorcache converts your prompt to a vector embedding
2. Searches for similar cached queries using cosine similarity
3. If similarity exceeds your threshold, returns the cached response
4. Otherwise, calls your LLM and caches the result

### What LLM providers do you support?

- OpenAI (GPT-4o, GPT-4o-mini, GPT-3.5-turbo)
- Anthropic (Claude 3.5 Sonnet, Haiku, Opus)
- Google (Gemini 1.5 Pro, Flash)
- More providers coming soon!

## Pricing & Plans

### How much does Vectorcache cost?

See our [Pricing page](pricing.md) for current plans. Free tier available for testing.

### Is there a free trial?

Yes! Sign up for a free account to test Vectorcache with your application.

### How do I calculate ROI?

```
Monthly Savings = (Cache Hits × Avg Query Cost) - Vectorcache Cost
ROI = (Monthly Savings / Vectorcache Cost) × 100%
```

See [Cost Optimization](../guides/cost-optimization.md) for detailed calculations.

## Technical

### What is similarity threshold?

The similarity threshold (0-1) determines how similar two queries must be for a cache hit. Higher values require closer matches. We recommend starting with 0.85.

See [Similarity Tuning](../guides/similarity-tuning.md) for details.

### How fast is Vectorcache?

- **Cache hit**: 50-150ms
- **Cache miss**: 1-5 seconds (includes LLM call)

Much faster than calling an LLM directly (typically 2-5 seconds).

### Do you store my LLM API keys?

Yes, your LLM API keys are encrypted at rest using AES-256 and never exposed in logs or responses. You can delete them anytime from the dashboard.

### Can I use my own embedding model?

Currently, Vectorcache uses optimized embedding models. Custom embedding models coming in a future release.

### What happens if Vectorcache is down?

Implement fallback logic in your application to call your LLM directly if Vectorcache is unavailable. See [Best Practices](../guides/best-practices.md#graceful-degradation).

## Data & Privacy

### Where is my data stored?

Data is stored in secure, SOC 2 compliant data centers in the US. EU data residency coming soon.

### Is my data encrypted?

Yes:
- **In transit**: TLS 1.3
- **At rest**: AES-256 encryption
- **API keys**: Separately encrypted

### Can I delete my cached data?

Yes, you can delete cache entries anytime from the dashboard or via API (coming soon).

### Do you train models on my data?

No, we never use your data to train models.

## Integration

### How long does integration take?

Most developers integrate Vectorcache in under 30 minutes:

1. Install SDK (1 minute)
2. Add API key (2 minutes)
3. Replace LLM calls (10-20 minutes)
4. Test and deploy (5-10 minutes)

### Do I need to change my existing code much?

Minimal changes required:

```typescript
// Before
const response = await openai.chat.completions.create({...});

// After
const result = await vectorcache.query({
  prompt: userMessage,
  model: 'gpt-4o'
});
```

### Can I use Vectorcache with streaming responses?

Streaming support coming in Q2 2025.

## Performance

### What's a good cache hit rate?

Depends on your use case:
- **Customer support**: 50-70%
- **Educational**: 60-80%
- **Documentation**: 40-60%
- **General chatbot**: 30-50%

### How can I improve my cache hit rate?

1. Lower similarity threshold (0.80-0.85)
2. Normalize user input
3. Use context segmentation
4. Group similar queries

See [Similarity Tuning](../guides/similarity-tuning.md).

### Does caching affect response quality?

When configured properly, no. Use higher similarity thresholds (0.90+) for use cases requiring exact matches.

## Troubleshooting

### Why am I getting low cache hit rates?

Common causes:
- Threshold too high (try 0.85)
- Queries are too unique
- Different contexts preventing matches
- Not enough cached data yet

### Why are my API calls failing?

Check:
- API key is valid and active
- LLM API keys configured in dashboard
- Request format is correct
- Not hitting rate limits

See [Error Handling](../api/errors.md).

### How do I debug cache misses?

Use debug mode:

```typescript
const result = await client.query({
  prompt: 'test',
  model: 'gpt-4o',
  includeDebug: true
});

console.log(result.debug);
```

## Limits

### What are the rate limits?

| Tier | Requests/Minute |
|------|----------------|
| Free | 100 |
| Pro | 1,000 |
| Enterprise | Custom |

### Is there a query size limit?

Maximum prompt size: 32,000 characters

### How many projects can I create?

- Free: 1 project
- Pro: 10 projects
- Enterprise: Unlimited

## Migration

### Can I migrate from another caching solution?

Yes! Contact support for migration assistance.

### How do I export my cached data?

Data export via API or dashboard coming soon.

## Support

### How do I get help?

- 📧 Email: support@vectorcache.com
- 💬 Discord: [Join our community](https://discord.gg/vectorcache)
- 📚 Docs: [docs.vectorcache.ai](https://docs.vectorcache.ai)
- 🐛 Issues: [GitHub](https://github.com/YOUR_USERNAME/vectorcache-docs/issues)

### What's your SLA?

- Free: Best effort
- Pro: 99.5% uptime
- Enterprise: 99.9% uptime with dedicated support

### Do you offer custom solutions?

Yes! Contact sales@vectorcache.com for enterprise and custom solutions.

## Still have questions?

[Contact our support team](support.md) - we're here to help!
