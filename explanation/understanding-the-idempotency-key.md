# Understanding the `Idempotency-Key`

`Idempotency-Key` helps prevent duplicate charges in payment systems. An `Idempotency-Key` is a unique UUID v4 value sent with each payment or refund request, ensuring the server processes it only once. Sending an `Idempotency-Key` in the request header with each request makes payments and refunds safer and more reliable. See [RFC 9562: Universally IDentifiers (UUIDs)](https://www.rfc-editor.org/rfc/rfc9562) for more information.

## When to use an `Idempotency-Key`

Use an `Idempotency-Key` request in the following situations:

* Payment creation (`POST /payments`)
* Refund creation (`POST /refunds`)
* Any operation where duplicate actions could cause issues

> **Note**: Use the `Idempotency-Key` for all `POST` requests in your payment integration to prevent duplicate charges.

## How idempotency works

* The first request with a key is processed.
* Subsequent requests with the same key return the cached response within the 24-hour window.
* `Idempotency-Key` expire 24 hours after their first use. The TTL (Time to Live) is controlled by the server-side infrastructure.

## Next steps

### How-to-guides

* [Preventing duplicate charges with the `Idempotency-Key`](./how-to-guides/preventing-duplicate-charges-with-the-idempotency-key.md): How to use `Idempotency-Key` to prevent duplicate charges.

### Reference

* [`Idempotency-Key`](../reference/Idempotency-Key.md): Covers the `Idempotency-Key` header format, requirements, expiration window, duplicate key behavior, and idempotency error codes.

---

Last updated: April 2026
