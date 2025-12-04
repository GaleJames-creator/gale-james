# How to process a refund

You can refund all or part of a payment. Refunds may take several days to appear on the customer's statement.

## Full refund example

```bash
curl https://sandbox-api..com/v1/refunds \
  -X POST \
  -H "Authorization: Bearer sk_test_1234567890abcdef" \
  -H "Content-Type: application/json" \
  -d '{
    "payment_id": "pay_1234567890abcdef",
    "reason": "requested_by_customer"
  }'
```

## Partial refund example

```bash
curl https://sandbox-api..com/v1/refunds \
  -X POST \
  -H "Authorization: Bearer sk_test_1234567890abcdef" \
  -H "Content-Type: application/json" \
  -d '{
    "payment_id": "pay_1234567890abcdef",
    "amount": 1000,
    "reason": "duplicate",
    "metadata": {
      "reason_detail": "Partial refund for shipping charge"
    }
  }'
```

### Refund reasons

Specify a reason when issuing a refund in the request body. Possible values are:

- `duplicate` - The payment was a duplicate charge
- `fraudulent` - The payment was a fraudulent transaction
- `requested_by_customer` - The customer requested a refund.

## Response

```json
{
  "id": "ref_1234567890abcdef",
  "object": "refund",
  "amount": 2999,
  "currency": "usd",
  "payment_id": "pay_1234567890abcdef",
  "status": "pending",
  "reason": "requested_by_customer",
  "created": 1699564800,
  "metadata": {}
}
```
