# Support

Get help with Vectorcache.

## Support Channels

### 📧 Email Support

**support@vectorcache.com**

- General questions
- Technical issues
- Account problems
- Response time: 24-48 hours

### 💬 Community Discord

**[Join our Discord](https://discord.gg/vectorcache)**

- Real-time chat with community
- Share tips and best practices
- Get quick answers from other users
- Connect with the team

### 📚 Documentation

**[docs.vectorcache.com](https://docs.vectorcache.com)**

- Complete API reference
- Step-by-step guides
- Code examples
- Best practices

### 🐛 Bug Reports

**[GitHub Issues](https://github.com/YOUR_USERNAME/vectorcache-docs/issues)**

- Report bugs
- Request features
- Track known issues
- Contribute to docs

## Support Tiers

### Free Tier
- Community Discord
- Documentation
- GitHub issues
- Email support (48-hour response)

### Pro Tier
- Priority email support (24-hour response)
- Discord priority channel
- Video call support (by appointment)

### Enterprise
- Dedicated support engineer
- 24/7 emergency support
- Custom SLA
- On-boarding assistance

## Common Issues

### Authentication Errors

**Problem**: `401 Unauthorized`

**Solution:**
1. Verify API key in dashboard
2. Check `Authorization` header format: `Bearer YOUR_KEY`
3. Ensure key hasn't been revoked

See [Authentication](../api/authentication.md)

### Low Cache Hit Rate

**Problem**: < 20% cache hit rate

**Solution:**
1. Lower similarity threshold (try 0.80-0.85)
2. Normalize user input
3. Check if queries are actually similar
4. Use context segmentation

See [Similarity Tuning](../guides/similarity-tuning.md)

### Rate Limit Errors

**Problem**: `429 Too Many Requests`

**Solution:**
1. Implement exponential backoff
2. Check rate limit headers
3. Upgrade to higher tier if needed

See [Error Handling](../api/errors.md)

### LLM Provider Errors

**Problem**: `502 Bad Gateway - LLM provider error`

**Solution:**
1. Verify LLM API keys in Settings → LLM Keys
2. Check LLM provider status
3. Ensure sufficient credits with provider

## Enterprise Support

For enterprise customers:

- **24/7 Support**: Phone, email, Slack
- **Dedicated Engineer**: Named support contact
- **Custom SLA**: 99.9% uptime guarantee
- **Priority Fixes**: Bug fixes within 24 hours
- **On-boarding**: Dedicated implementation support

Contact [sales@vectorcache.com](mailto:sales@vectorcache.com)

## Status Page

Check service status: **[status.vectorcache.com](https://status.vectorcache.com)**

Subscribe to updates:
- Email notifications
- SMS alerts (Enterprise)
- Slack integration (Enterprise)

## Office Hours

Join our weekly office hours:

**Every Wednesday at 2 PM PST**
- Ask questions live
- Demo your use case
- Get expert advice
- Meet the team

[Join office hours →](https://meet.vectorcache.com/office-hours)

## Feature Requests

Have an idea? We want to hear it!

1. **Check existing requests**: [GitHub Discussions](https://github.com/YOUR_USERNAME/vectorcache/discussions)
2. **Upvote**: Vote for features you want
3. **Submit new**: Create a new discussion

Popular requests often ship within weeks!

## Security Issues

**DO NOT** report security issues publicly.

Email: **security@vectorcache.com**

We'll respond within 24 hours and work with you to:
- Understand the issue
- Develop a fix
- Coordinate disclosure

## Feedback

We love feedback!

- Product feedback: feedback@vectorcache.com
- Documentation: Suggest edits via GitHub
- General thoughts: support@vectorcache.com

## Contact Sales

For enterprise, custom solutions, or partnerships:

📧 **sales@vectorcache.com**

We can help with:
- Enterprise pricing
- Custom deployment
- Volume discounts
- Integration support
- Training and on-boarding

---

**Need help right now?**

[Email Support](mailto:support@vectorcache.com) | [Join Discord](https://discord.gg/vectorcache) | [View Docs](/)
