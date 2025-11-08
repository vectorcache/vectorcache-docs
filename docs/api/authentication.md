# Authentication

Learn how to authenticate your API requests to Vectorcache.

## API Keys

Vectorcache uses API keys for authentication. Each API key is associated with a specific project.

### Creating an API Key

1. Log in to your [dashboard](https://app.vectorcache.com)
2. Navigate to your project
3. Go to the **API Keys** tab
4. Click **Create API Key**
5. Give it a descriptive name
6. Copy the key immediately - you won't see it again!

### API Key Format

API keys follow this format:

```
vc_1234567890abcdef1234567890abcdef
```

- Prefix: `vc_`
- 32 hexadecimal characters

## Using API Keys

### HTTP Header

Include your API key in the `Authorization` header using Bearer authentication:

```
Authorization: Bearer vc_1234567890abcdef1234567890abcdef
```

### Example Requests

=== "cURL"

    ```bash
    curl -X POST "https://api.vectorcache.ai/v1/cache/query" \
      -H "Authorization: Bearer vc_1234567890abcdef1234567890abcdef" \
      -H "Content-Type: application/json" \
      -d '{
        "prompt": "What is AI?",
        "model": "gpt-4o"
      }'
    ```

=== "JavaScript"

    ```javascript
    const response = await fetch('https://api.vectorcache.ai/v1/cache/query', {
      method: 'POST',
      headers: {
        'Authorization': 'Bearer vc_1234567890abcdef1234567890abcdef',
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        prompt: 'What is AI?',
        model: 'gpt-4o'
      })
    });
    ```

=== "Python"

    ```python
    import requests

    headers = {
        'Authorization': 'Bearer vc_1234567890abcdef1234567890abcdef',
        'Content-Type': 'application/json'
    }

    response = requests.post(
        'https://api.vectorcache.ai/v1/cache/query',
        json={'prompt': 'What is AI?', 'model': 'gpt-4o'},
        headers=headers
    )
    ```

## Security Best Practices

### 1. Store Keys Securely

❌ **Never** hardcode API keys:

```javascript
// DON'T DO THIS
const apiKey = 'vc_1234567890abcdef1234567890abcdef';
```

✅ Use environment variables:

```javascript
// DO THIS
const apiKey = process.env.VECTORCACHE_API_KEY;
```

### 2. Use Environment Variables

**.env file:**
```bash
VECTORCACHE_API_KEY=vc_1234567890abcdef1234567890abcdef
```

**Never commit .env files to git:**
```bash
# .gitignore
.env
.env.local
```

### 3. Rotate Keys Regularly

- Create a new API key
- Update your application to use the new key
- Revoke the old key once migration is complete

### 4. Use Different Keys per Environment

```bash
# .env.development
VECTORCACHE_API_KEY=vc_dev_key123...

# .env.production
VECTORCACHE_API_KEY=vc_prod_key456...
```

### 5. Limit Key Exposure

- Use separate keys for different applications
- Revoke keys when they're no longer needed
- Monitor key usage in the dashboard

## Managing API Keys

### Viewing API Keys

1. Go to your project dashboard
2. Navigate to **API Keys** tab
3. View all active and revoked keys

You'll see:
- Key prefix (e.g., `vc_1234...`)
- Key name
- Creation date
- Last used date
- Status (Active/Revoked)

### Revoking API Keys

To revoke a key:

1. Go to **API Keys** tab
2. Find the key to revoke
3. Click **Revoke**
4. Confirm the action

⚠️ **Warning**: Revoking a key immediately disables it. Any applications using that key will receive 401 errors.

### Key Usage Tracking

Monitor your API key usage:

- **Last Used**: When the key was last used
- **Request Count**: Total requests made with this key
- **Usage Metrics**: Available in the Analytics tab

## Authentication Errors

### 401 Unauthorized

Missing or invalid API key:

```json
{
  "detail": "Invalid or missing API key"
}
```

**Causes:**
- API key not provided
- Invalid API key format
- Key has been revoked
- Key doesn't exist

**Solution:**
- Verify your API key is correct
- Check if the key is active in your dashboard
- Ensure the `Authorization` header is properly formatted

### 403 Forbidden

API key doesn't have permission:

```json
{
  "detail": "API key does not have permission to access this project"
}
```

**Causes:**
- Using an API key from a different project
- Key permissions have been restricted

**Solution:**
- Use the correct API key for this project
- Check key permissions in the dashboard

## API Key Scopes (Coming Soon)

Future releases will support scoped API keys with granular permissions:

- `cache:read` - Read from cache only
- `cache:write` - Write to cache only
- `cache:*` - Full cache access
- `analytics:read` - View analytics
- `keys:manage` - Manage API keys

## Next Steps

- [Cache Endpoints](cache.md) - Use the cache API
- [Error Handling](errors.md) - Handle authentication errors
- [Best Practices](../guides/best-practices.md) - Production security tips
