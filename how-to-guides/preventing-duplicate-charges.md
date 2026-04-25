# Preventing duplicate charges with `Idempotency-Key`

Duplicate charges frustrate customers and create support issues. The  `Idempotency-Key` ensures each payment is processed only once, even if the same request is sent multiple times due to network issues or retries. This guide explains how `Idempotency-Key` work, why they matter, and how to use them effectively in your payment system.

## Prerequisites

Before you start, make sure you:

* Have an integrated payment system.
* Are familiar with APIs.
* Understand HTTP status codes.

## Step 1. Generate the `Idempotency-Key`

Generate a UUID v4 value and store it in a Python variable. This value becomes your `Idempotency-Key` header value.

```python
import uuid
import requests

idempotency_key = str(uuid.uuid4())

response = requests.post(
    "https://sandbox-api.payment.com/v1/payments",
    headers={
        "Authorization": f"Bearer {YOUR_API_KEY}",  # Never log API keys.
        "Content-Type": "application/json",
        # idempotency_key is your Python variable
        # "Idempotency-Key" is the required HTTP header name
        "Idempotency-Key": idempotency_key
    },
    json=payment_data
)
```

Store the `idempotency_key` value — you'll reuse it in Step 3 for retries.

## Step 2. Send the key with your request

Add an `Idempotency-Key` header with the unique `idempotency_key` you got in step 1 when sending a POST request to create or refund a payment.

> **Note**: Replace `{YOUR_API_KEY}` with your API key. Never hardcode API keys in your code. Also note, the card number `4242424242424242` only works in sandbox environments. The example charges $29.99 with the Payments API.

```bash
curl https://sandbox-api.payment.com/v1/payments \
  -X POST \
  -H "Authorization: Bearer {YOUR_API_KEY}" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: {YOUR_IDEMPOTENCY_KEY}" \
  -d '{
    "amount": 2999,
    "currency": "usd",
    "payment_method": {
      "type": "card",
      "card": {
        "number": "4242424242424242",
        "exp_month": 12,
        "exp_year": 2025,
        "cvc": "123"
      }
    },
    "customer": {
      "email": "customer@example.com"
    }
  }'
```

### Step 3. Reuse the same key on retries

To reuse the `idempotency_key`, store the key from step 1 and use it on retry.

```python
# Store the key separately from the request body
stored_idempotency_key = idempotency_key

# On retry, reuse the stored key
response = requests.post(
    "https://sandbox-api.payment.com/v1/payments",
    headers={
        "Authorization": f"Bearer {api_key}",
        "Content-Type": "application/json",
        "Idempotency-Key": stored_idempotency_key
    },
    json=payment_data
)
```

## Next steps

### Explanation

* [Understanding the `Idempotency-Key`](../explanation/understanding-the-idempotency-key.md): Covers the key webhook concepts, how they work, and when to use them.

### Reference

* [`Idempotency-Key`](../reference/Idempotency-Key): Covers the `Idempotency-Key`  header format, requirements, expiration window, duplicate key behavior, and idempotency error codes.

## Tutorials

* [Getting started with payments API](../tutorials/payment-api-getting-started.md): How to authenticate, navigate test vs. production environments, and make your first payment call in 15 minutes.

---

Last updated: April 2026
