# Manage rate limits

Use these steps to prevent and manage rate limits in your integration.

## Prerequisites

Before you start, make sure you:

- Have an integrated payment system
- Are familiar with the Payment API
- Understand HTTP status codes

## Steps

- Monitor rate limit headers in each response.
- Use exponential backoff for retries.
- Cache GET requests where possible, such as for payments or webhook lists.
- Batch operations when applicable.
- Request a tier upgrade if you consistently reach the rate limit.


## Retry logic

The following example demonstrates how to retry a failed operation using exponential backoff. Use this pattern to handle temporary errors, such as rate limiting, by waiting and retrying.

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

## Next steps

- [Managing API rate limits](../reference/payment-api-rate-limits.md)
- [Implement retry logic for failed payments](./payment-retry-logic.md)
