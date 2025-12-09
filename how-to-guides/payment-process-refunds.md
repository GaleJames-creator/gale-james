# How to process a refund

This guide shows you, step by step, how to issue a full or partial refund using the refunds API endpoint. Use this guide when you need to return funds to a customer, either in full or in part. Note that refunds may take several days to appear on the customer’s statement.

## Prerequisites

- You have a valid API key.

- You know the `payment_id` of the transaction to refund.

Follow these steps to send a refund request using the refunds endpoint.

## Step 1: Send a refund

Use the refunds endpoint to send a full or partial refund. You must include the transaction's `payment_id` and a refund `reason` in the request body. Accepted values are:

- `duplicate` - The payment was a duplicate charge
- `fraudulent` - The payment was a fraudulent transaction
- `requested_by_customer` - The customer requested a refund.

### Full refund example

To refund the entire payment, use this `curl` command:

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

### Partial refund example

To refund part of a payment, specify the `amount` (in cents) in your request:

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

## Step 2: Check the API response

A successful refund returns a response like this:

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
