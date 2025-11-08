# cURL Examples

Use Vectorcache directly with cURL - no SDK required.

## Basic Query

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

## With Context

```bash
curl -X POST "https://api.vectorcache.ai/v1/cache/query" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Explain photosynthesis",
    "context": "educational-biology",
    "model": "gpt-4o",
    "similarity_threshold": 0.85
  }'
```

## With Debug Information

```bash
curl -X POST "https://api.vectorcache.ai/v1/cache/query" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "What is quantum computing?",
    "model": "gpt-4o",
    "similarity_threshold": 0.85,
    "include_debug": true
  }'
```

## Using Environment Variables

Store your API key in an environment variable:

```bash
export VECTORCACHE_API_KEY="your_api_key_here"

curl -X POST "https://api.vectorcache.ai/v1/cache/query" \
  -H "Authorization: Bearer $VECTORCACHE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "What is AI?",
    "model": "gpt-4o"
  }'
```

## Pretty Print JSON Response

Use `jq` to format the response:

```bash
curl -X POST "https://api.vectorcache.ai/v1/cache/query" \
  -H "Authorization: Bearer $VECTORCACHE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "What is machine learning?",
    "model": "gpt-4o"
  }' | jq '.'
```

## Save Response to File

```bash
curl -X POST "https://api.vectorcache.ai/v1/cache/query" \
  -H "Authorization: Bearer $VECTORCACHE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Explain deep learning",
    "model": "gpt-4o"
  }' -o response.json
```

## Extract Specific Fields

Extract only the response text:

```bash
curl -X POST "https://api.vectorcache.ai/v1/cache/query" \
  -H "Authorization: Bearer $VECTORCACHE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "What is AI?",
    "model": "gpt-4o"
  }' | jq -r '.response'
```

Check if it was a cache hit:

```bash
curl -X POST "https://api.vectorcache.ai/v1/cache/query" \
  -H "Authorization: Bearer $VECTORCACHE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "What is AI?",
    "model": "gpt-4o"
  }' | jq -r '.cache_hit'
```

## Batch Queries Script

Process multiple prompts:

```bash
#!/bin/bash

PROMPTS=(
  "What is machine learning?"
  "Explain deep learning"
  "What are neural networks?"
)

for prompt in "${PROMPTS[@]}"; do
  echo "Querying: $prompt"
  curl -s -X POST "https://api.vectorcache.ai/v1/cache/query" \
    -H "Authorization: Bearer $VECTORCACHE_API_KEY" \
    -H "Content-Type: application/json" \
    -d "{
      \"prompt\": \"$prompt\",
      \"model\": \"gpt-4o\",
      \"similarity_threshold\": 0.85
    }" | jq '{cache_hit, response: .response[:100]}'
  echo "---"
done
```

## Error Handling

Show HTTP status code and response:

```bash
curl -i -X POST "https://api.vectorcache.ai/v1/cache/query" \
  -H "Authorization: Bearer $VECTORCACHE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "What is AI?",
    "model": "gpt-4o"
  }'
```

Only show status code:

```bash
curl -s -o /dev/null -w "%{http_code}" \
  -X POST "https://api.vectorcache.ai/v1/cache/query" \
  -H "Authorization: Bearer $VECTORCACHE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "What is AI?",
    "model": "gpt-4o"
  }'
```

## Retry on Failure

Simple retry script:

```bash
#!/bin/bash

MAX_RETRIES=3
RETRY_DELAY=2

for i in $(seq 1 $MAX_RETRIES); do
  response=$(curl -s -w "\n%{http_code}" \
    -X POST "https://api.vectorcache.ai/v1/cache/query" \
    -H "Authorization: Bearer $VECTORCACHE_API_KEY" \
    -H "Content-Type: application/json" \
    -d '{
      "prompt": "What is AI?",
      "model": "gpt-4o"
    }')

  http_code=$(echo "$response" | tail -n1)
  body=$(echo "$response" | head -n-1)

  if [ "$http_code" = "200" ]; then
    echo "$body" | jq '.'
    exit 0
  fi

  echo "Attempt $i failed with code $http_code"
  if [ $i -lt $MAX_RETRIES ]; then
    echo "Retrying in ${RETRY_DELAY}s..."
    sleep $RETRY_DELAY
  fi
done

echo "Max retries exceeded"
exit 1
```

## Testing Different Thresholds

Test various similarity thresholds:

```bash
#!/bin/bash

THRESHOLDS=(0.70 0.80 0.85 0.90 0.95)

for threshold in "${THRESHOLDS[@]}"; do
  echo "Testing threshold: $threshold"
  curl -s -X POST "https://api.vectorcache.ai/v1/cache/query" \
    -H "Authorization: Bearer $VECTORCACHE_API_KEY" \
    -H "Content-Type: application/json" \
    -d "{
      \"prompt\": \"What is machine learning?\",
      \"model\": \"gpt-4o\",
      \"similarity_threshold\": $threshold
    }" | jq '{threshold: '"$threshold"', cache_hit, similarity_score}'
  echo "---"
done
```

## Response Format

### Success Response (Cache Hit)

```json
{
  "cache_hit": true,
  "response": "Machine learning is a subset of artificial intelligence...",
  "similarity_score": 0.92,
  "cost_saved": 0.003,
  "llm_provider": "cache"
}
```

### Success Response (Cache Miss)

```json
{
  "cache_hit": false,
  "response": "Machine learning is a subset of artificial intelligence...",
  "similarity_score": null,
  "cost_saved": 0,
  "llm_provider": "openai"
}
```

### Error Response

```json
{
  "detail": "Invalid API key"
}
```

## Common HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 400 | Bad Request - Invalid parameters |
| 401 | Unauthorized - Invalid API key |
| 429 | Too Many Requests - Rate limited |
| 500 | Internal Server Error |

## Advanced Usage

### Measure Response Time

```bash
curl -w "\nTime: %{time_total}s\n" \
  -X POST "https://api.vectorcache.ai/v1/cache/query" \
  -H "Authorization: Bearer $VECTORCACHE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "What is AI?",
    "model": "gpt-4o"
  }' | jq '.'
```

### Parallel Requests

Use GNU Parallel to send concurrent requests:

```bash
parallel -j 5 "curl -s -X POST 'https://api.vectorcache.ai/v1/cache/query' \
  -H 'Authorization: Bearer $VECTORCACHE_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{\"prompt\": \"What is AI?\", \"model\": \"gpt-4o\"}' | jq '.cache_hit'" \
  ::: {1..10}
```

## Next Steps

- [JavaScript SDK](javascript.md) - Full SDK with TypeScript support
- [Python SDK](python.md) - Python implementation
- [API Reference](../api/overview.md) - Complete API documentation
