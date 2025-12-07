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

