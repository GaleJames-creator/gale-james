# Prevent duplicate charges with idempotency keys

Use these steps to add idempotency keys to your payment requests and prevent duplicate charges.

## Prerequisites

Before you start, make sure you:

- Have an integrated payment system
- Are familiar with APIs
- Understand HTTP status codes

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

### Create a payment with idempotency

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

## Next steps

### Reference

- [Preventing duplicate charges with idempotency keys](../reference/payment-idempotency-keys.md)

---

Last updated: April 2026
