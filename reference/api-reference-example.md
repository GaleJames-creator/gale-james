# API reference example

## Essential cURL commands

### Create a payment
```bash
curl https://sandbox-api..com/v1/payments \
  -X POST \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"amount": 2999, "currency": "usd", "payment_method": {...}}'
```

### Retrieve a payment
```bash
curl https://sandbox-api..com/v1/payments/PAYMENT_ID \
  -H "Authorization: Bearer YOUR_API_KEY"
```

### Create a refund
```bash
curl https://sandbox-api..com/v1/refunds \
  -X POST \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"payment_id": "PAYMENT_ID", "reason": "requested_by_customer"}'
```

### List webhooks
```bash
curl https://sandbox-api..com/v1/webhooks \
  -H "Authorization: Bearer YOUR_API_KEY"
```

### Key concepts checklist

- [ ] Verify API keys are stored securely in environment variables
- [ ] Use the Sandbox environment for testing
- [ ] Handle errors using the correct HTTP status codes
- [ ] Set up idempotency for payment requests
- [ ] Keep an eye on rate limit headers
- [ ] Check webhook signatures
- [ ] Log payment IDs so you can reconcile later
- [ ] Try different test cards to check both successful and declined payments
