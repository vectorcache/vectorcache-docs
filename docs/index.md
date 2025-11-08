# Vectorcache Documentation

Welcome to Vectorcache - the intelligent semantic caching layer for LLM applications.

## What is Vectorcache?

Vectorcache is an AI-powered caching solution that uses semantic similarity to cache and retrieve LLM responses. Instead of exact-match caching, Vectorcache understands the *meaning* of queries, dramatically improving cache hit rates and reducing API costs.

## Key Features

- **🎯 Semantic Matching** - Uses vector embeddings to match similar queries, not just identical ones
- **💰 Cost Reduction** - Save up to 90% on LLM API costs with intelligent caching
- **⚡ Fast Response Times** - Serve cached responses in milliseconds instead of seconds
- **🔒 Secure & Private** - Your data is encrypted and isolated per project
- **🛠 Easy Integration** - Drop-in SDK for JavaScript/TypeScript and Python
- **📊 Analytics Dashboard** - Track cache performance, costs, and usage metrics

## Quick Example

=== "JavaScript/TypeScript"

    ```typescript
    import { VectorcacheClient } from 'vectorcache';

    const client = new VectorcacheClient({
      apiKey: 'your_api_key',
      baseUrl: 'https://api.vectorcache.ai'
    });

    const result = await client.query({
      prompt: 'What is machine learning?',
      model: 'gpt-4o',
      similarityThreshold: 0.85
    });

    console.log(`Cache hit: ${result.cache_hit}`);
    console.log(`Response: ${result.response}`);
    ```

=== "Python"

    ```python
    import requests

    headers = {
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json"
    }

    data = {
        "prompt": "What is machine learning?",
        "model": "gpt-4o",
        "similarity_threshold": 0.85
    }

    response = requests.post(
        "https://api.vectorcache.ai/v1/cache/query",
        json=data,
        headers=headers
    )

    result = response.json()
    print(f"Cache hit: {result['cache_hit']}")
    ```

=== "cURL"

    ```bash
    curl -X POST "https://api.vectorcache.ai/v1/cache/query" \
      -H "Authorization: Bearer YOUR_API_KEY" \
      -H "Content-Type: application/json" \
      -d '{
        "prompt": "What is machine learning?",
        "model": "gpt-4o",
        "similarity_threshold": 0.85
      }'
    ```

## How It Works

1. **Query Submission** - Your application sends a prompt to Vectorcache
2. **Semantic Search** - Vectorcache searches for semantically similar cached queries
3. **Cache Hit/Miss** - Returns cached response if similarity exceeds threshold, otherwise calls your LLM
4. **Cost Savings** - Track savings and performance in real-time dashboard

## Getting Started

<div class="grid cards" markdown>

- :material-clock-fast:{ .lg .middle } **Quick Start**

    ---

    Get up and running in 5 minutes

    [:octicons-arrow-right-24: Quick Start Guide](getting-started/quickstart.md)

- :material-package-variant:{ .lg .middle } **Installation**

    ---

    Install the SDK for your platform

    [:octicons-arrow-right-24: Installation Guide](getting-started/installation.md)

- :material-book-open-variant:{ .lg .middle } **API Reference**

    ---

    Complete API documentation

    [:octicons-arrow-right-24: API Docs](api/overview.md)

- :material-frequently-asked-questions:{ .lg .middle } **FAQ**

    ---

    Common questions and answers

    [:octicons-arrow-right-24: View FAQ](about/faq.md)

</div>

## Use Cases

- **Customer Support Chatbots** - Cache common questions and responses
- **Educational Platforms** - Reduce costs for frequently asked educational queries
- **Documentation Search** - Serve similar documentation queries from cache
- **Content Generation** - Cache similar content requests
- **Data Analysis** - Reuse responses for similar analytical queries

## Why Vectorcache?

Traditional caching only works for *exact* matches. If a user asks "What is ML?" after someone asked "What is machine learning?", traditional caching misses. Vectorcache understands these are the same question and serves the cached response.

**Result**: 5-10x higher cache hit rates compared to traditional caching.

## Support

Need help? We're here for you:

- 📧 Email: support@vectorcache.com
- 💬 Discord: [Join our community](https://discord.gg/vectorcache)
- 🐛 Issues: [GitHub Issues](https://github.com/YOUR_USERNAME/vectorcache-docs/issues)

---

Ready to reduce your LLM costs? [Get started now →](getting-started/quickstart.md)
