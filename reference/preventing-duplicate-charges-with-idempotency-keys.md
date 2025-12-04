# Preventing duplicate charges with idempotency keys

Idempotency ensures you do not incur duplicate charges when retrying a request.

## When to use idempotency

Use idempotent requests in the following situations:

- Payment creation (`POST /payments`)
- Refund creation (`POST /refunds`)
- Any operation where duplicate actions could cause issues

## How to use idempotency keys

Add an `Idempotency-Key` header with a unique value, such as a UUID:

```bash
curl https://sandbox-api.payment.com/v1/payments \
  -X POST \
  -H "Authorization: Bearer sk_test_1234567890abcdef" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: a1b2c3d4-e5f6-7890-abcd-ef1234567890" \
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

### How idempotency works

1. The first request with a specific idempotency key is processed normally.
2. Subsequent requests with the same key return the cached response.
3. Idempotency keys expire after 24 hours.

### Implementation example

```javascript
const { v4: uuidv4 } = require('uuid');

async function createPayment(paymentData) {
  const idempotencyKey = uuidv4();
  
  const response = await fetch('https://sandbox-api.payment.com/v1/payments', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.PAYMENT_API_KEY}`,
      'Content-Type': 'application/json',
      'Idempotency-Key': idempotencyKey
    },
    body: JSON.stringify(paymentData)
  });
  
  return response.json();
}
```
