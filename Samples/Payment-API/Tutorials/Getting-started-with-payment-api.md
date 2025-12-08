# Getting started with Payments API

This tutorial shows you how to integrate payment processing into an application using the Payments API. By following these steps, you will process a test payment.

## What you'll learn

* [How to authenticate with an API](#step-1-get-your-api-keys)
* [How to choose an environment](#step-2-choose-your-environment)
* [How to send your first test payment](#step-3-send-your-first-payments-api-request)
* [How to check payment status](#step-4-check-the-payment-status)
* [How to handle a simple error](#step-5-handle-a-simple-error)
* [How to retrieve the payment details](#step-6-retrieve-the-payment-details)

## Prerequisites

* Ensure you have a Payment account
* Know REST APIs and JSON
* Have a development environment with command-line access
* A cURL or HTTP client.

**Estimated time to complete:** 10 minutes

## Step 1: Get your API keys

API keys authenticate your requests to Payment. You'll need different keys for testing and production.

### Obtain your keys

1. Sign in to the Payment Dashboard.
2. Navigate to **Developers** → **API Keys**.
3. You will find two types of API keys:
   - **Test keys** (prefix: `sk_test_`): Use these for development and testing purposes.
   - **Live keys** (prefix: `sk_live_`): Use these for production after all testing is complete.

### Protect your API keys

**Security best practices:**

* Do not share API keys in code (add them to `.gitignore`).
* Store keys in environment variables or a secure configuration system. 
* Never include keys in public code or repositories.
* Rotate keys regularly, especially if compromised.

Rotate API keys regularly and monitor for issues. For help, contact your Customer Success Manager.

**Setting an environment variable**

To set your API key, choose one of the commands below:

```bash
# Linux/macOS
export PAYMENT_API_KEY="sk_test_1234567890abcdef"

# Windows (Command Prompt)
set PAYMENT_API_KEY=sk_test_1234567890abcdef

# Windows (PowerShell)
$env:PAYMENT_API_KEY="sk_test_1234567890abcdef"
```

## Step 2: Choose your environment

The Payments API provides two environments: sandbox (for development and testing) and production environment (for live payments).

| Environment | Base URL | Purpose |
|-------------|----------|---------|
| **Sandbox** | `https://sandbox-api.payment.com/v1` | Testing and development |
| **Production** | `https://api.payment.com/v1` | Live payment processing |

**Note**: This guide uses the sandbox environment and test API keys.

### Test cards

The following table lists sample test card numbers, each demonstrating a different transaction behavior for testing in the sandbox environment.

| Test Card           | Card Type  | Behavior |
|---------------------|------------|----------|
| 4242 4242 4242 4242 | Visa       | Payment succeeds |
| 4000 0000 0000 0002 | Visa       | Card declined (generic) |
| 4000 0000 0000 9995 | Visa       | Card declined (insufficient funds) |
| 5555 5555 5555 4444 | Mastercard | Payment succeeds |

For test cards, use any future expiration date, a 3-digit CVC, and any ZIP code.

## Step 3: Send your first Payments API request

Use cURL to make a test payment. The example charges $29.99 with the Payments API.

### Request Example

```bash
curl https://sandbox-api.payment.com/v1/payments \
  -X POST \
  -H "Authorization: Bearer sk_test_1234567890abcdef" \
  -H "Content-Type: application/json" \
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
      "email": "customer@example.com",
      "name": "Jane Doe"
    },
    "description": "Test payment for order #12345"
  }'
```

### Understanding the request

The request includes the following:

**Headers:**
- `Authorization: Bearer sk_test_...` - Your API key for authentication
- `Content-Type: application/json` - Indicates the request body is JSON

**Request body parameters:**
- `amount` - Payment amount in cents (`2999` = $29.99)
- `currency` - Three-letter ISO lowercase currency code (e.g., `usd`)
- `payment_method` - Card information
- `customer` - Customer information for receipts and records
- `description` - Optional note that appears on statements

## Step 4: Check the payment status

When you receive the response your Payments API request, verify `status` field displays `succeeded`. If the requests fails, see [Step 5: Handle a simple error](#tep-5-handle-a-simple-error) for details.

### Successful response

If the payment succeeds, the response is similar to the following:

```json
{
  "id": "pay_1234567890abcdef",
  "object": "payment",
  "amount": 2999,
  "currency": "usd",
  "status": "succeeded",
  "payment_method": {
    "type": "card",
    "card": {
      "brand": "visa",
      "last4": "4242",
      "exp_month": 12,
      "exp_year": 2025
    }
  },
  "customer": {
    "email": "customer@example.com",
    "name": "Jane Doe"
  },
  "description": "Test payment for order #12345",
  "created": 1699564800,
  "receipt_url": "https://payment.com/receipts/pay_1234567890abcdef"
}
```

**Key fields:**
- `id` - Unique payment ID. Save this ID for refunds or retrievals.
- `status` - Payment result. Possible values: `succeeded`, `failed`, `pending`, or `canceled`.
- `receipt_url` - URL to the customer receipt.


## Step 5: Handle a simple error

Payments may fail for various reasons. The following describes how to respond to common issues:

### Declined card example

The following example demonstrates a declined card. Use declined card tests to verify your system handles failures and displays error messages to users.

```bash
curl https://sandbox-api.payment.com/v1/payments \
  -X POST \
  -H "Authorization: Bearer sk_test_1234567890abcdef" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 2999,
    "currency": "usd",
    "payment_method": {
      "type": "card",
      "card": {
        "number": "4000000000000002",
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

### Error response (HTTP 402)

A payment failure due to a declined card returns an HTTP `402` error. The following is a typical error response:

```json
{
  "error": {
    "type": "payment_error",
    "code": "card_declined",
    "message": "Your card was declined.",
    "decline_code": "generic_decline"
  }
}
```

### Common error types

During payment processing, you may encounter the following common errors:

| HTTP Status | Error Type | Description | Action |
|-------------|-----------|-------------|---------|
| `400` | `invalid_request_error` | Invalid parameters | Check your request format |
| `401` | `authentication_error` | Invalid API key | Verify your API key |
| `402` | `payment_error` | Payment failed | Display error to customer |
| `404` | `invalid_request_error` | Resource not found | Check the ID in your request |
| `429` | `rate_limit_error` | Too many requests | Implement retry logic with backoff |
| `500` | `api_error` | Server error | Retry the request |

If you get an error, see [Best practices for handling API errors](../Reference/Best-practices-for-handling-payment-api-errors.md) for instructions.

## Step 6: Retrieve the payment details

Retrieve payment details using the payment ID.

### Request example

```bash
curl https://sandbox-api.payment.com/v1/payments/pay_1234567890abcdef \
  -H "Authorization: Bearer sk_test_1234567890abcdef"
```

### Response

A successful request returns transaction details, including payment status, amount, payment method, and customer information.

```json
{
  "id": "pay_1234567890abcdef",
  "object": "payment",
  "amount": 2999,
  "currency": "usd",
  "status": "succeeded",
  "payment_method": {
    "type": "card",
    "card": {
      "brand": "visa",
      "last4": "4242",
      "exp_month": 12,
      "exp_year": 2025
    }
  },
  "customer": {
    "email": "customer@example.com",
    "name": "Jane Doe"
  },
  "description": "Test payment for order #12345",
  "created": 1699564800,
  "receipt_url": "https://payment.com/receipts/pay_1234567890abcdef"
}
```

### Using caching

The GET endpoint supports caching to improve performance. The response includes the following headers:

- `Cache-Control` header: Caching directives
- `ETag` header: Resource version identifier

**Conditional request example:**

Use the `If-None-Match` header to return data only if the resource has changed. This reduces unnecessary API requests and bandwidth.

```bash
curl https://sandbox-api.payment.com/v1/payments/pay_1234567890abcdef \
  -H "Authorization: Bearer sk_test_1234567890abcdef" \
  -H 'If-None-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"'
```

If the resource is unchanged, the server returns `304 Not Modified` with no response body.

## What you've accomplished

You have completed the following steps:

✅ Authenticated with your API<br>
✅ How to choose an environment<br>
✅ Processed a test payment<br>
✅ Hoe to check payment status<br>
✅ How to andle a simple error<br>
✅ How to retrieve the payment details  

## Next steps
Ready for more? Check out:

- [How to implement retry logic for failed payments](../How-to-guides/How-to-implement-retry-logic-for-failed-payments.md)
- [How to processes refunds](../How-to-guides/How-to-process-refunds.md)


### Going to production

To process live payments, complete the following steps:

1. **Complete your account verification** in the Dashboard
2. **Switch to live API keys** (prefix: `sk_live_`)
3. **Update your base URL** to `https://api..com/v1`
4. **Review security practices** - Ensure keys are stored securely
5. **Set up monitoring** - Track payment success rates and errors
6. **[Configure webhooks](../../Webhooks/Tutorials/Creating-a-webhook.md)** for production URLs
7. **Test with small amounts** before processing larger transactions

### Key concepts checklist

- [ ] Verify API keys are stored securely in environment variables
- [ ] Use the sandbox environment for testing
- [ ] Handle errors using the correct HTTP status codes
- [ ] Set up idempotency for payment requests
- [ ] Keep an eye on rate limit headers
- [ ] Check webhook signatures
- [ ] Log payment IDs so you can reconcile later
- [ ] Try different test cards to check both successful and declined payments

