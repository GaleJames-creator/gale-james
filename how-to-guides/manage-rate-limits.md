# Manage rate limits

Use these steps to prevent and manage rate limits in your integration.

# Manage API errors

## Prerequisites

Before you start, make sure you:

- Have an integrated payment system
- Are familiar with the Payment API
- Understand HTTP status codes

## Preventing rate limits

- Monitor rate limit headers in each response.
- Use exponential backoff for retries.
- Cache GET requests where possible, such as for payments or webhook lists.
- Batch operations when applicable.
- Request a tier upgrade if you consistently reach the rate limit.
