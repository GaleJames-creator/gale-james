# `Idempotency-Key`

This reference covers the `Idempotency-Key` header format, requirements, expiration window, duplicate key behavior, and idempotency error codes.

## Header name and format

The `Idempotency-Key: {uuid-v4}` is an HTTP header containing a randomly generated version 4 Universally Unique Identifier (UUID). The header ensures POST payment requests are processed only once. If another POST payment request uses the header again, you would see an error in the response. This assures that a customer isn’t charged twice.

The key must conform to the UUID v4 standard (randomly generated). Avoid using sensitive information (like user emails) as seeds for the key. Use cryptographic randomness to prevent information leakage.

## Requirements

A new UUID v4 must be generated for every request that creates, updates, or deletes a payment.

## Expiration window

The server must temporarily store the keys (typically with a 24-hour retention period) to identify and ignore replayed requests. You can retry during the 24-hour window if you send the exact same request body as the first request.

## Duplicate key behavior

The server handles duplicate keys differently depending on the request:

* **Same key, same body**: the server returns the cached response from the original request. No duplicate charge occurs.
* **Same key, different body**: the server returns a 422 error. You cannot reuse a key with different parameters.
* **Same key, still processing**: the server returns a 409 error. Wait for the original request to complete before retrying.

## Error types

Use the following table to identify your error code and find the appropriate resolution path.

| HTTP Status | Error Type                    | Description        | Action                                      |
| ----------- | ----------------------------- | ------------------ | ------------------------------------------- |
| `400`       | `missing_idempotency_key`     | Missing key        | Provide the required header and try again.  |
| `409`       | `idempotency_conflict`        | Concurrent request | A request with this key is already running. |
| `422`       | `idempotency_replay_mismatch` | Payload mismatch   | Reusing key with different parameters.      |

> **Note**: A `202 Accepted` response indicates the payment is still being processed. Poll the payment status endpoint to verify completion.

## Next steps

### Explanation

* [Understanding the `Idempotency-Key`](../explanation/understanding-the-idempotency-key.md): Covers the idempotency concepts, how they work, and when to use them.

### How-to-guides

* [Preventing duplicate charges with the `Idempotency-Key`](./how-to-guides/preventing-duplicate-charges-with-the-idempotency-key.md): How to use `Idempotency-Key` to prevent duplicate charges.

---

Last updated: April 2026
