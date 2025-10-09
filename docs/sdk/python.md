# Python SDK

Complete guide to using Vectorcache with Python (REST API approach).

!!! note "Python SDK Coming Soon"
    A dedicated Python SDK package is in development. Currently, use the REST API with `requests` or `httpx`.

## Installation

```bash
pip install requests
# or for async support
pip install httpx
```

## Quick Start

```python
import requests
import os

api_key = os.environ.get('VECTORCACHE_API_KEY')
base_url = 'https://api.vectorcache.com'

headers = {
    'Authorization': f'Bearer {api_key}',
    'Content-Type': 'application/json'
}

data = {
    'prompt': 'What is machine learning?',
    'model': 'gpt-4o',
    'similarity_threshold': 0.85
}

response = requests.post(
    f'{base_url}/v1/cache/query',
    json=data,
    headers=headers
)

result = response.json()
print(f"Response: {result['response']}")
print(f"Cache hit: {result['cache_hit']}")
```

## Using requests

### Basic Query

```python
import requests
from typing import Dict, Any

def query_vectorcache(
    prompt: str,
    model: str,
    api_key: str,
    similarity_threshold: float = 0.85,
    context: str = None
) -> Dict[str, Any]:
    """Query the Vectorcache API."""

    url = 'https://api.vectorcache.com/v1/cache/query'

    headers = {
        'Authorization': f'Bearer {api_key}',
        'Content-Type': 'application/json'
    }

    data = {
        'prompt': prompt,
        'model': model,
        'similarity_threshold': similarity_threshold
    }

    if context:
        data['context'] = context

    response = requests.post(url, json=data, headers=headers)
    response.raise_for_status()

    return response.json()

# Usage
result = query_vectorcache(
    prompt='Explain quantum computing',
    model='gpt-4o',
    api_key=os.environ['VECTORCACHE_API_KEY']
)

print(result)
```

### Error Handling

```python
import requests

def query_with_error_handling(prompt: str, model: str, api_key: str):
    url = 'https://api.vectorcache.com/v1/cache/query'

    headers = {
        'Authorization': f'Bearer {api_key}',
        'Content-Type': 'application/json'
    }

    data = {
        'prompt': prompt,
        'model': model,
        'similarity_threshold': 0.85
    }

    try:
        response = requests.post(url, json=data, headers=headers, timeout=30)
        response.raise_for_status()
        return response.json()

    except requests.exceptions.HTTPError as e:
        if e.response.status_code == 401:
            raise ValueError("Invalid API key")
        elif e.response.status_code == 429:
            raise ValueError("Rate limit exceeded")
        elif e.response.status_code == 500:
            raise ValueError(f"Server error: {e.response.text}")
        else:
            raise ValueError(f"HTTP error: {e}")

    except requests.exceptions.Timeout:
        raise ValueError("Request timed out")

    except requests.exceptions.RequestException as e:
        raise ValueError(f"Request failed: {e}")
```

## Using httpx (Async)

### Async Client

```python
import httpx
import asyncio
from typing import Dict, Any

async def query_vectorcache_async(
    prompt: str,
    model: str,
    api_key: str,
    similarity_threshold: float = 0.85
) -> Dict[str, Any]:
    """Async query to Vectorcache API."""

    url = 'https://api.vectorcache.com/v1/cache/query'

    headers = {
        'Authorization': f'Bearer {api_key}',
        'Content-Type': 'application/json'
    }

    data = {
        'prompt': prompt,
        'model': model,
        'similarity_threshold': similarity_threshold
    }

    async with httpx.AsyncClient(timeout=30.0) as client:
        response = await client.post(url, json=data, headers=headers)
        response.raise_for_status()
        return response.json()

# Usage
async def main():
    result = await query_vectorcache_async(
        prompt='What is AI?',
        model='gpt-4o',
        api_key=os.environ['VECTORCACHE_API_KEY']
    )
    print(result)

asyncio.run(main())
```

### Batch Processing

```python
import httpx
import asyncio
from typing import List, Dict, Any

async def process_batch(
    prompts: List[str],
    model: str,
    api_key: str
) -> List[Dict[str, Any]]:
    """Process multiple prompts concurrently."""

    async with httpx.AsyncClient(timeout=30.0) as client:
        tasks = [
            query_with_client(client, prompt, model, api_key)
            for prompt in prompts
        ]
        return await asyncio.gather(*tasks)

async def query_with_client(
    client: httpx.AsyncClient,
    prompt: str,
    model: str,
    api_key: str
) -> Dict[str, Any]:
    url = 'https://api.vectorcache.com/v1/cache/query'

    headers = {
        'Authorization': f'Bearer {api_key}',
        'Content-Type': 'application/json'
    }

    data = {
        'prompt': prompt,
        'model': model,
        'similarity_threshold': 0.85
    }

    response = await client.post(url, json=data, headers=headers)
    response.raise_for_status()
    return response.json()

# Usage
async def main():
    prompts = [
        'What is machine learning?',
        'Explain deep learning',
        'What are neural networks?'
    ]

    results = await process_batch(
        prompts,
        model='gpt-4o',
        api_key=os.environ['VECTORCACHE_API_KEY']
    )

    cache_hits = sum(1 for r in results if r['cache_hit'])
    print(f"Cache hit rate: {cache_hits / len(results) * 100:.1f}%")

asyncio.run(main())
```

## Vectorcache Client Class

Create a reusable client class:

```python
import requests
from typing import Optional, Dict, Any
from dataclasses import dataclass

@dataclass
class VectorcacheResponse:
    cache_hit: bool
    response: str
    similarity_score: Optional[float]
    cost_saved: float
    llm_provider: str

class VectorcacheClient:
    def __init__(self, api_key: str, base_url: str = 'https://api.vectorcache.com'):
        self.api_key = api_key
        self.base_url = base_url
        self.session = requests.Session()
        self.session.headers.update({
            'Authorization': f'Bearer {api_key}',
            'Content-Type': 'application/json'
        })

    def query(
        self,
        prompt: str,
        model: str,
        similarity_threshold: float = 0.85,
        context: Optional[str] = None,
        include_debug: bool = False
    ) -> VectorcacheResponse:
        """Query the cache."""

        url = f'{self.base_url}/v1/cache/query'

        data = {
            'prompt': prompt,
            'model': model,
            'similarity_threshold': similarity_threshold,
            'include_debug': include_debug
        }

        if context:
            data['context'] = context

        response = self.session.post(url, json=data, timeout=30)
        response.raise_for_status()

        result = response.json()
        return VectorcacheResponse(
            cache_hit=result['cache_hit'],
            response=result['response'],
            similarity_score=result.get('similarity_score'),
            cost_saved=result['cost_saved'],
            llm_provider=result['llm_provider']
        )

    def close(self):
        """Close the session."""
        self.session.close()

    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.close()

# Usage
with VectorcacheClient(api_key=os.environ['VECTORCACHE_API_KEY']) as client:
    result = client.query(
        prompt='What is machine learning?',
        model='gpt-4o',
        similarity_threshold=0.85
    )

    print(f"Response: {result.response}")
    print(f"Cache hit: {result.cache_hit}")
    if result.cache_hit:
        print(f"Similarity: {result.similarity_score}")
        print(f"Saved: ${result.cost_saved}")
```

## Framework Integration

### FastAPI

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import os

app = FastAPI()
client = VectorcacheClient(api_key=os.environ['VECTORCACHE_API_KEY'])

class QueryRequest(BaseModel):
    prompt: str
    model: str = 'gpt-4o'
    similarity_threshold: float = 0.85

@app.post("/query")
async def query_cache(request: QueryRequest):
    try:
        result = client.query(
            prompt=request.prompt,
            model=request.model,
            similarity_threshold=request.similarity_threshold
        )
        return result
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

### Django

```python
# views.py
from django.http import JsonResponse
from django.views import View
import json
import os

class CacheQueryView(View):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.client = VectorcacheClient(
            api_key=os.environ['VECTORCACHE_API_KEY']
        )

    def post(self, request):
        try:
            data = json.loads(request.body)
            result = self.client.query(
                prompt=data['prompt'],
                model=data.get('model', 'gpt-4o'),
                similarity_threshold=data.get('similarity_threshold', 0.85)
            )
            return JsonResponse({
                'cache_hit': result.cache_hit,
                'response': result.response,
                'cost_saved': result.cost_saved
            })
        except Exception as e:
            return JsonResponse({'error': str(e)}, status=500)
```

### Flask

```python
from flask import Flask, request, jsonify
import os

app = Flask(__name__)
client = VectorcacheClient(api_key=os.environ['VECTORCACHE_API_KEY'])

@app.route('/query', methods=['POST'])
def query():
    try:
        data = request.json
        result = client.query(
            prompt=data['prompt'],
            model=data.get('model', 'gpt-4o'),
            similarity_threshold=data.get('similarity_threshold', 0.85)
        )
        return jsonify({
            'cache_hit': result.cache_hit,
            'response': result.response,
            'cost_saved': result.cost_saved
        })
    except Exception as e:
        return jsonify({'error': str(e)}), 500
```

## Examples

### Chatbot with Metrics

```python
class ChatbotWithCache:
    def __init__(self, api_key: str):
        self.client = VectorcacheClient(api_key)
        self.total_queries = 0
        self.cache_hits = 0
        self.total_saved = 0.0

    def chat(self, message: str) -> str:
        result = self.client.query(
            prompt=message,
            model='gpt-4o',
            context='customer-support',
            similarity_threshold=0.85
        )

        self.total_queries += 1
        if result.cache_hit:
            self.cache_hits += 1
            self.total_saved += result.cost_saved

        return result.response

    def get_stats(self) -> Dict[str, Any]:
        hit_rate = (self.cache_hits / self.total_queries * 100
                   if self.total_queries > 0 else 0)
        return {
            'total_queries': self.total_queries,
            'cache_hits': self.cache_hits,
            'hit_rate': f'{hit_rate:.1f}%',
            'total_saved': f'${self.total_saved:.4f}'
        }

# Usage
bot = ChatbotWithCache(api_key=os.environ['VECTORCACHE_API_KEY'])

print(bot.chat('What is machine learning?'))
print(bot.chat('Explain ML'))  # Similar query, likely cache hit
print(bot.get_stats())
```

### Retry Logic

```python
import time
from typing import Optional

def query_with_retry(
    client: VectorcacheClient,
    prompt: str,
    model: str,
    max_retries: int = 3,
    backoff_factor: float = 2.0
) -> Optional[VectorcacheResponse]:
    """Query with exponential backoff retry."""

    for attempt in range(max_retries):
        try:
            return client.query(prompt=prompt, model=model)
        except requests.exceptions.HTTPError as e:
            if e.response.status_code == 429:  # Rate limit
                if attempt < max_retries - 1:
                    delay = min(backoff_factor ** attempt, 10)
                    print(f"Rate limited, retrying in {delay}s...")
                    time.sleep(delay)
                    continue
            raise
        except requests.exceptions.Timeout:
            if attempt < max_retries - 1:
                print(f"Timeout, retrying... (attempt {attempt + 1})")
                continue
            raise

    return None
```

## Best Practices

1. **Use environment variables** for API keys
2. **Implement proper error handling** with try/except blocks
3. **Set timeouts** on all requests (default: 30 seconds)
4. **Use connection pooling** with `requests.Session()`
5. **Implement retry logic** for transient failures
6. **Monitor performance** - track cache hit rates
7. **Use async** (`httpx`) for high-throughput applications

## Next Steps

- [Configuration Guide](../getting-started/configuration.md)
- [API Reference](../api/overview.md)
- [Best Practices](../guides/best-practices.md)
