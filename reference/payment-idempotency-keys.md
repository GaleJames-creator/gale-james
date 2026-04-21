# Preventing duplicate charges with idempotency keys

Duplicate charges can happen in payment systems when network issues or timeouts cause clients to retry. Idempotency keys make sure each payment is processed only once, even if the request is repeated.

## When to use idempotency

Use idempotent requests in the following situations:

- Payment creation (`POST /payments`)
- Refund creation (`POST /refunds`)
- Any operation where duplicate actions could cause issues

## How idempotency works

- The first request with a specific idempotency key is processed normally.
- Subsequent requests with the same key return the cached response.
- Idempotency keys expire after 24 hours.

---

Last updated: April 2026
