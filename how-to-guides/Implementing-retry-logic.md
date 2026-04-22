# Implementing retry logic for failed payments

This guide shows you how to set up automatic retries for failed payments due to temporary issues, such as network errors or rate limits, in your payment system.

## Prerequisites

Before you start, make sure you:

* Have an integrated payment system.
* Are familiar with the Payment API.
* Understand HTTP status codes.

## Step 1: Decide when to retry a payment

Retry a payment only if you receive one of the following error codes:

* `429` (rate limits)
* `500`, `502`, `503`, `504` (server errors)
* Network timeouts

**Do not retry if you receive these errors:**
  
* `400`: Invalid request. Fix the request instead and notify the customer about the issue.
* `401`: Authentication error. Check your API key and inform the customer about the unsuccessful authentication attempt.
* `402`: Card declined. Customer action required; surface this error to the customer so they can resolve it directly with their bank or card provider.

## Step 2: Implement exponential backoff for retries

Automatically retry a failed payment using exponential backoff. This approach helps avoid overloading the API and effectively handles temporary issues.

Adding random jitter prevents multiple clients from retrying simultaneously, which would overwhelm the API.

The following example retries the payment up to three times. Wait at least 1, 2, or 4 seconds before retrying. If you get a `200` response, the payment succeeded. For errors `429`, `500`, `502`, `503`, `504`, or a timeout, wait 1, 2, or 4 seconds before retrying. For other errors, including `400`, `401`, and `402`, stop and raise an exception.

### Python function with retry logic example
  
```python
import time
import requests
import random

def create_payment_with_retry(payment_data, api_key, max_retries=3):
    for attempt in range(max_retries):
        try:
            response = requests.post(
                "https://api.example.com/v1/payments",
                json=payment_data,
                headers={"Authorization": f"Bearer {api_key}"}  # Never log API keys.
            )
            
            if response.status_code == 200:
                return response.json()
            
            if response.status_code in [429, 500, 502, 503, 504]:
                wait_time = (2 ** attempt) + random.uniform(0, 1)
                time.sleep(wait_time)
                continue
            
            # Don't retry other errors
            response.raise_for_status()
            
        except requests.exceptions.Timeout:
            if attempt < max_retries - 1:
                time.sleep((2 ** attempt) + random.uniform(0, 1))  # Exponential backoff with jitter
                continue
            raise
    
    raise Exception("Max retries exceeded")
```

## Step 3: Add idempotency to prevent duplicate charges

Always use the same `idempotency_key` for each retry of the same payment. This prevents duplicate charges if the request is processed multiple times.

```python
headers={
    "Authorization": f"Bearer {api_key}",  # Never log API keys.
    "Idempotency-Key": payment_data.get("idempotency_key", "")
}
```

> **Note**: Load the api_key from an environmental variable:

```python
import os
api_key = os.getenv("PAYMENT_API_KEY")
```

## Step 4: Log retry attempts for monitoring

Track each retry attempt in your logs. This can help you monitor issues and optimize your retry strategy.

```python
import logging

logging.info(f"Payment retry attempt {attempt + 1}/{max_retries}")
```

## Next steps

### Explanation

* [Authorization vs. capture: understanding the difference](../explanation/payment-authorization-vs-capture.md)

### How-to guides

* [Managing API errors](../how-to-guides/managing-api-errors.md)

### Reference

* [Preventing duplicate charges with idempotency keys](../reference/payment-idempotency-keys.md): Covers how idempotency keys work and why they are essential for preventing accidental duplicate charges.
* [Payment API rate limits](../reference/payment-api-rate-limits.md): Covers rate limit tiers and headers, and handling rate limits.
* [Payment decline codes](../reference/decline-codes.md): Complete reference of all authorization decline codes with hard/soft classification.

---

Last updated: April 2026
