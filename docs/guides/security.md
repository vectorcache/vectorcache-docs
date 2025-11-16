# Security

Best practices for securing your Vectorcache integration.

## API Key Security

### Store Keys Securely

✅ **Do:**
- Use environment variables
- Store in secret management systems (AWS Secrets Manager, HashiCorp Vault)
- Use `.env` files (never commit to git)

❌ **Don't:**
- Hardcode in source code
- Commit to version control
- Share in public channels

### Rotate Keys Regularly

Implement key rotation:

```typescript
async function rotateAPIKey() {
  // 1. Create new API key in dashboard
  const newKey = await createNewKey('Production Key v2');

  // 2. Update environment variable
  process.env.VECTORCACHE_API_KEY = newKey;

  // 3. Test new key
  await testAPIKey(newKey);

  // 4. Revoke old key
  await revokeOldKey(oldKeyId);
}
```

## Data Privacy

### Sensitive Data Handling

Don't cache sensitive information:

```typescript
function shouldCache(prompt: string): boolean {
  // Don't cache prompts with PII
  const sensitivePatterns = [
    /\b\d{3}-\d{2}-\d{4}\b/, // SSN
    /\b\d{16}\b/,            // Credit card
    /\b[\w\.-]+@[\w\.-]+\.\w+\b/ // Email
  ];

  return !sensitivePatterns.some(pattern => pattern.test(prompt));
}

if (shouldCache(userPrompt)) {
  await client.query({ prompt: userPrompt, model: 'gpt-4o' });
} else {
  await directLLMCall(userPrompt);
}
```

### Data Encryption

Vectorcache implements comprehensive encryption to protect your data:

- **In transit**: TLS 1.3 for all API communications
- **At rest**: Fernet symmetric encryption (AES-128 in CBC mode) for all sensitive cache data
  - Customer prompts are encrypted before storage
  - LLM responses are encrypted before storage
  - Context data is encrypted before storage
  - Embeddings remain unencrypted (required for semantic similarity search)
- **API keys**: Encrypted using PBKDF2-derived keys with 100,000 iterations
- **Performance**: Encryption adds negligible overhead (~0.05ms per operation, <0.2% of total API latency)
- **Backward compatibility**: Automatically handles existing unencrypted cache entries during transition

**What's encrypted:**
- ✅ Customer prompts (prompt_text)
- ✅ LLM responses (response_text)
- ✅ Context data (context_text)
- ✅ LLM API keys (user_llm_keys)
- ❌ Vector embeddings (required for similarity search)
- ❌ Metadata and timestamps

**Encryption method:**
- Algorithm: Fernet (symmetric encryption)
- Key derivation: PBKDF2-HMAC-SHA256 with 100,000 iterations
- Cipher: AES-128 in CBC mode with HMAC authentication

## Access Control

### Environment Separation

Use separate keys per environment:

```bash
# Development
VECTORCACHE_API_KEY=vc_dev_...

# Staging
VECTORCACHE_API_KEY=vc_staging_...

# Production
VECTORCACHE_API_KEY=vc_prod_...
```

### Least Privilege

Grant minimum necessary permissions (coming soon):

```typescript
// Future feature: Scoped API keys
const readOnlyKey = createAPIKey({
  name: 'Analytics Dashboard',
  scopes: ['cache:read', 'analytics:read']
});
```

## Network Security

### Use HTTPS Only

Always use HTTPS:

```typescript
const client = new VectorcacheClient({
  apiKey: process.env.VECTORCACHE_API_KEY!,
  baseUrl: 'https://api.vectorcache.ai' // Always HTTPS
});
```

### IP Whitelisting (Enterprise)

For enterprise customers:

```typescript
// Configure in dashboard
allowedIPs: [
  '203.0.113.0/24',
  '198.51.100.0/24'
]
```

## Compliance

### GDPR Compliance

- Right to be forgotten: Delete user cache entries
- Data portability: Export user data
- Consent: Don't cache without user consent

### SOC 2 Compliance

Vectorcache is SOC 2 compliant:
- Annual audits
- Security monitoring
- Incident response procedures

## Security Checklist

- [ ] API keys stored in environment variables
- [ ] `.env` files in `.gitignore`
- [ ] Separate keys per environment
- [ ] Regular key rotation (90 days)
- [ ] No sensitive data in cache
- [ ] HTTPS only
- [ ] Monitor for suspicious activity
- [ ] Incident response plan in place

## Next Steps

- [Best Practices](best-practices.md) - General best practices
- [API Reference](../api/authentication.md) - Authentication docs
