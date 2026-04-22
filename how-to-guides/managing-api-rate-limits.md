# Managing rate limits

Use these steps to prevent and manage rate limits in your integration.

## Prerequisites

Before you start, make sure you:

* Have an integrated payment system.
* Are familiar with APIs.
* Understand HTTP status codes.

## Steps

1. Monitor rate limit headers in each response.
2. Use exponential backoff for retries.
3. Cache GET requests where possible, such as retrieving webhook lists or payment history, rather than re-fetching on every request.
4. Batch operations, when applicable, such as retrieving multiple payment records in a single request rather than multiple calls.
5. Request a tier upgrade if you consistently reach the rate limit.

## Retry logic

The following example demonstrates how to retry a failed operation using exponential backoff. Use this pattern to handle temporary errors, such as rate limiting, by waiting and retrying.

```javascript
const sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms));

async function makeRequestWithRetry(url, options, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    const response = await fetch(url, options);
    
    if (response.status === 429) {
      if (attempt === maxRetries - 1) {
        throw new Error('Max retries exceeded after rate limiting');
      }
      const retryAfter = response.headers.get('Retry-After') || 60;
      const delay = Math.pow(2, attempt) * 1000; // Exponential backoff
      await sleep(Math.max(delay, retryAfter * 1000));
      continue;
    }

  const retryAfter = parseInt(response.headers.get('Retry-After')) || 60;
  const delay = Math.pow(2, attempt) * 1000;
  await sleep(Math.max(delay, retryAfter * 1000));
  continue;
}
    
    return response;
  }
  
  throw new Error('Max retries exceeded');
}
```

Monitor these headers in each API response:

* `X-RateLimit-Remaining` — requests remaining in current window
* `X-RateLimit-Reset` — timestamp when the limit resets
* `Retry-After` — seconds to wait after a 429 response

## Next steps

### How-to guides

* [Implementing retry logic for failed payments](./implementing-retry-logic.md): How to implement automatic retries for failed payments.

---

Last updated: April 2026
