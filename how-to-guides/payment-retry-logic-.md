# Implement retry logic for failed payments

This guide shows you how to set up automatic retries for failed payments due to temporary issues, such as network errors or rate limits, in your payment system.

## What you'll need

Before you start, make sure you:

- Have an integrated payment system

- Are familiar with the Payment API

- Understand HTTP status codes

## Step 1: Decide when to retry a payment

Retry a payment only if you receive one of the following error codes:

- 429 (rate limits)

- 500, 502, 503, 504 (server errors)

- Network timeouts

**Do not retry if you receive these errors:**
  
- `400`: Invalid request. Fix the request instead and notify the user about the issue.
- `401`: Authentication error. Check your API key and inform the user about the unsuccessful authentication attempt.
- `402`: Card declined. User action required; surface this error to the user so they can resolve it directly with their bank or card provider.

## Step 2: Implement Exponential Backoff for Retries

Automatically retry failed payment requests using exponential backoff. This approach helps avoid overloading the API and effectively handles temporary issues.

The following example retries the payment request up to three times. If you get a 200 response, the payment succeeded. For errors 429, 500, 502, 503, 504, or a timeout, wait 1, 2, or 4 seconds before retrying. For other errors, stop and raise an exception.

If all attempts fail, raise an exception. This method handles temporary issues and prevents API overload.

### Python function with retry logic example
  
```python
import time
import requests

def create_payment_with_retry(payment_data, max_retries=3):
    for attempt in range(max_retries):
        try:
            response = requests.post(
                "https://api.example.com/v1/payments",
                json=payment_data,
                headers={"Authorization": "Bearer {API_KEY}"} # Handle API keys securely in logs.
            )
            
            if response.status_code == 200:
                return response.json()
            
            if response.status_code in [429, 500, 502, 503, 504]:
                wait_time = 2 ** attempt  # Wait 1s, 2s, 
                time.sleep(wait_time)
                continue
            
            # Don't retry other errors
            response.raise_for_status()
            
        except requests.exceptions.Timeout:
            if attempt < max_retries - 1:
                time.sleep(2 ** attempt) # Incremental wait on retry
                continue
            # Stop retrying after max_retries attempts
            raise
    
    raise Exception("Max retries exceeded")
```

## Step 3: Add idempotency to prevent duplicate charges

Always use the same idempotency key for each retry of the same payment. This prevents duplicate charges if the request is processed multiple times.

```python
import uuid

payment_data = {
    "amount": 1000,
    "currency": "USD",
    "idempotency_key": str(uuid.uuid4())
}
```

## Step 4: Log retry attempts for monitoring

Track each retry attempt in your logs. This can help you monitor issues and optimize your retry strategy.

```python
import logging

logging.info(f"Payment retry attempt {attempt + 1}/{max_retries}")
```
