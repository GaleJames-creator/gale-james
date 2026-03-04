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

## Next steps

- [Managing API rate limits](../reference/payment-api-rate-limits.md)
