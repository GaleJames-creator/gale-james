# How to implement retry logic for failed payments

Failed payments can lead to revenue loss and erode user trust. Use this guide to set up automatic retries for failed payments due to temporary issues, such as network errors or rate limits.

## Prerequisites

Before you proceed, ensure you have an integrated payment system, are familiar with the Payment API, and understand HTTP status codes.

## When to retry

Retry payments only when you encounter these error codes: 429 (rate limits), 500, 502, 503, and 504 (server errors), or network timeouts.
  
**Don't retry:**
  
- `400`: Invalid request. Fix the request instead and notify the user about the issue.
- `401`: Authentication error. Check your API key and inform the user about the unsuccessful authentication attempt.
- `402`: Card declined. User action required; surface this error to the user so they can resolve it directly with their bank or card provider.

## Implement exponential backoff

We recommend automatically retrying payment requests that fail due to rate limits, server errors, or network timeouts. Use exponential backoff to avoid overloading the API.

The following example retries the payment request up to three times. If you get a `200` response, the payment worked. For errors `429`, `500`, `502`, `503`, `504`, or a timeout, wait 1, 2, or 4 seconds before retrying. For any other error, stop and raise an exception.

If all attempts fail, raise an exception. This method handles temporary issues and prevents API overload.
  
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

## Add idempotency

Use idempotency keys to prevent duplicate charges.
  
```python
import uuid

payment_data = {
    "amount": 1000,
    "currency": "USD",
    "idempotency_key": str(uuid.uuid4())
}
```

**Important**: Always use the same idempotency key for each retry of the same payment. Save and reuse this key to prevent duplicate charges.

## Log retry attempts

Track each retry attempt for monitoring.
  
```python
import logging

logging.info(f"Payment retry attempt {attempt + 1}/{max_retries}")
```

## Related guides
- How to handle card declines
- How to implement webhooks for payment status
- How to configure rate limit handling
