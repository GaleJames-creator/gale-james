# Rate limits

The Payment API uses rate limits to maintain API stability and ensure fair usage.

### Rate limit tiers

| Tier | Requests per Minute | Use Case |
|------|---------------------|----------|
| Standard | 100 | Most applications |
| Premium | 1,000 | High-volume merchants |

## Rate limit headers

Each API response includes your current rate limit details.

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1699564860
```

- `X-RateLimit-Limit`: Maximum requests allowed per minute.
- `X-RateLimit-Remaining`: Requests remaining in current window.
- `X-RateLimit-Reset`: Unix timestamp when the limit resets.

## Handling rate limits

If you go over the rate limit, the API returns a `429 Too Many Requests` response.

```json
{
  "error": {
    "type": "rate_limit_error",
    "code": "rate_limit_exceeded",
    "message": "You have exceeded your rate limit. Please retry after 60 seconds."
  }
}
```

### Best practices for managing rate limits

1. **Monitor rate limit headers** in each response.
2. **Use exponential backoff** for retries.
3. **Cache GET requests** where possible, such as for payments or webhooks lists.
4. **Batch operations** when applicable.
5. **Request a tier upgrade** if you consistently reach the rate limit.

### Retry logic (pseudocode) example

The following pseudocode demonstrates how to retry a failed operation using exponential backoff. Use this pattern to handle temporary errors, such as rate limiting, by waiting and retrying.

```javascript
async function makeRequestWithRetry(url, options, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    const response = await fetch(url, options);
    
    if (response.status === 429) {
      const retryAfter = response.headers.get('Retry-After') || 60;
      const delay = Math.pow(2, attempt) * 1000; // Exponential backoff
      await sleep(Math.max(delay, retryAfter * 1000));
      continue;
    }
    
    return response;
  }
  
  throw new Error('Max retries exceeded');
}
```
