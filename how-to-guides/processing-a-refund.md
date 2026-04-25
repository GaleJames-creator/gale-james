# Processing a refund

This guide shows you, step by step, how to issue a full or partial refund using the refunds API endpoint. Use this guide when you need to return funds to a customer, either in full or in part.

> **Note**: Refunds may take 3-5 business days to appear on the customer’s statement.

## Prerequisites

* You have a valid API key.
* You know the `payment_id` of the transaction to refund.

## Step 1: Create a refund request

Use the refunds endpoint to send a full or partial refund. You must include the transaction's `payment_id` and a refund `reason` in the request body. Accepted values are:

| Reason                  | Description                               |
|-------------------------|-------------------------------------------|
| `duplicate`             | The payment was a duplicate charge.       |
| `fraudulent`            | The payment was a fraudulent transaction. |
| `requested_by_customer` | The customer requested a refund.          |

### Full refund example

To refund the entire payment, use this `curl` command:

> **Note**: Replace `{YOUR_API_KEY}` with your API key. Never hardcode API keys in your code. The sandbox URL `sandbox-api.com` is for testing only — replace with your production URL for live transactions.

```bash
curl https://sandbox-api.com/v1/refunds \
  -X POST \
  -H "Authorization: Bearer {YOUR_API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{
    "payment_id": "pay_1234567890abcdef",
    "reason": "requested_by_customer"
  }'
```

### Partial refund example

To refund part of a payment, specify the `amount` (in cents) in your request:

```bash
curl https://sandbox-api.com/v1/refunds \
  -X POST \
  -H "Authorization: Bearer {YOUR_API_KEY}" \
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
  "amount": 1000,
  "currency": "usd",
  "payment_id": "pay_1234567890abcdef",
  "status": "pending",
  "reason": "requested_by_customer",
  "created": 1699564800,
  "metadata": {}
}
```

A successful response returns a `200 OK` status code.

## Next steps

### Reference

* [API error codes](../reference/api-error-codes.md): Quick reference table of common API error codes and resolution paths.
* [API reference guide](../reference/api-reference-guide.md): Covers three API endpoints: retrieve available payment options, update payment options, and create an order refund.

## Tutorials

* [Getting started with payments API](../tutorials/payment-api-getting-started.md): How to authenticate, navigate test vs. production environments, and make your first payment call in 15 minutes.

---

Last updated: April 2026
