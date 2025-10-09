# Quick Start Guide

Get started with Vectorcache in under 5 minutes.

## Step 1: Create an Account

1. Visit [https://app.vectorcache.com](https://app.vectorcache.com)
2. Sign up with your email or GitHub account
3. Verify your email address

## Step 2: Create a Project

1. Log in to your dashboard
2. Click **"Create New Project"**
3. Give your project a name (e.g., "My Chatbot")
4. Click **"Create Project"**

## Step 3: Add Your LLM API Keys

Before caching can work, Vectorcache needs your LLM provider API keys:

1. Navigate to **Settings → LLM Keys**
2. Click **"Add LLM Key"**
3. Select your provider (OpenAI, Anthropic, etc.)
4. Enter your API key
5. Give it a name for reference

!!! note "Security"
    Your LLM API keys are encrypted at rest and never exposed in logs or responses.

## Step 4: Get Your Vectorcache API Key

1. Go to your project dashboard
2. Navigate to the **API Keys** tab
3. Click **"Create API Key"**
4. Give it a name (e.g., "Production Key")
5. **Copy the API key immediately** - you won't see it again!

## Step 5: Install the SDK

=== "JavaScript/TypeScript"

    ```bash
    npm install vectorcache-js
    ```

=== "Python"

    ```bash
    pip install vectorcache-python
    ```

## Step 6: Make Your First Request

=== "JavaScript/TypeScript"

    ```typescript
    import { VectorcacheClient } from 'vectorcache-js';

    const client = new VectorcacheClient({
      apiKey: 'your_api_key_here',
      baseUrl: 'https://api.vectorcache.com'
    });

    async function main() {
      const result = await client.query({
        prompt: 'Explain quantum computing in simple terms',
        context: 'Educational content for beginners',
        model: 'gpt-4o',
        similarityThreshold: 0.85
      });

      console.log(`Cache hit: ${result.cache_hit}`);
      console.log(`Response: ${result.response}`);

      if (result.cost_saved) {
        console.log(`Cost saved: $${result.cost_saved}`);
      }
    }

    main();
    ```

=== "Python"

    ```python
    import requests

    api_key = "your_api_key_here"
    base_url = "https://api.vectorcache.com"

    headers = {
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json"
    }

    data = {
        "prompt": "Explain quantum computing in simple terms",
        "context": "Educational content for beginners",
        "model": "gpt-4o",
        "similarity_threshold": 0.85
    }

    response = requests.post(
        f"{base_url}/v1/cache/query",
        json=data,
        headers=headers
    )

    result = response.json()

    print(f"Cache hit: {result['cache_hit']}")
    print(f"Response: {result['response']}")

    if result.get('cost_saved'):
        print(f"Cost saved: ${result['cost_saved']}")
    ```

## Understanding the Response

Your first request will be a **cache miss** since nothing is cached yet:

```json
{
  "cache_hit": false,
  "response": "Quantum computing is...",
  "similarity_score": null,
  "cost_saved": 0,
  "llm_provider": "openai"
}
```

The second request with a similar prompt will be a **cache hit**:

```json
{
  "cache_hit": true,
  "response": "Quantum computing is...",
  "similarity_score": 0.92,
  "cost_saved": 0.003,
  "llm_provider": "cache"
}
```

## Next Steps

✅ You're now caching LLM responses!

Here's what to explore next:

- [**Tune Similarity Threshold**](../guides/similarity-tuning.md) - Optimize cache hit rates
- [**View Analytics**](../guides/best-practices.md#monitoring) - Track savings in your dashboard
- [**API Reference**](../api/overview.md) - Explore all available endpoints
- [**Best Practices**](../guides/best-practices.md) - Production deployment tips

## Troubleshooting

### Cache Always Missing

- Ensure `similarity_threshold` is not too high (try 0.80-0.85)
- Verify your prompts are actually similar
- Check that you're using the same `model` parameter

### Authentication Errors

- Confirm your API key is correctly formatted
- Check that the key hasn't been revoked
- Ensure you're using the correct project API key

### LLM API Errors

- Verify your LLM API keys are valid in Settings → LLM Keys
- Check that you have sufficient credits with your LLM provider
- Ensure the `model` parameter matches your provider (e.g., 'gpt-4o' for OpenAI)

Need more help? [Contact support](../about/support.md)
