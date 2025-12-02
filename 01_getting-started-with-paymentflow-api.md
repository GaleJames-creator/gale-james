# Getting started with PaymentFlow API

This guide describes how to integrate PaymentFlow payment processing into your application. By following these steps, you will process a test payment.

## Prerequisites

Ensure you have a PaymentFlow account, know REST APIs and JSON, have a development environment with command-line access, and a cURL or HTTP client.

**Estimated time to complete:** 10 minutes

---

## Step 1: Get your API keys

API keys authenticate your requests to PaymentFlow. You'll need different keys for testing and production.

### Obtain your keys

1. Sign in to the PaymentFlow Dashboard.
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
export PAYMENTFLOW_API_KEY="sk_test_1234567890abcdef"

# Windows (Command Prompt)
set PAYMENTFLOW_API_KEY=sk_test_1234567890abcdef

# Windows (PowerShell)
$env:PAYMENTFLOW_API_KEY="sk_test_1234567890abcdef"
```

---

## Step 2: Choose your environment

PaymentFlow provides two environments: sandbox (for development and testing) and production environment (for live payments).

| Environment | Base URL | Purpose |
|-------------|----------|---------|
| **Sandbox** | `https://sandbox-api.paymentflow.com/v1` | Testing and development |
| **Production** | `https://api.paymentflow.com/v1` | Live payment processing |

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

---

## Step 3: Send your first API request

Use cURL to make a test payment. The example charges $29.99 with the payments API.

### Request Example

```bash
curl https://sandbox-api.paymentflow.com/v1/payments \
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
- `amount` - Payment amount in cents (2999 = $29.99)
- `currency` - Three-letter ISO lowercase currency code (e.g., usd)
- `payment_method` - Card information
- `customer` - Customer information for receipts and records
- `description` - Optional note that appears on statements

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
  "receipt_url": "https://paymentflow.com/receipts/pay_1234567890abcdef"
}
```

**Key fields:**
- `id` - Unique payment ID. Save this ID for refunds or retrievals.
- `status` - Payment result. Possible values: `succeeded`, `failed`, `pending`, or `canceled`.
- `receipt_url` - URL to the customer receipt.

---

## Step 4: Handle errors

Payments may fail for various reasons. The following describes how to respond to common issues:

### Declined card example

The following example demonstrates a declined card. Use declined card tests to verify your system handles failures and displays error messages to users.

```bash
curl https://sandbox-api.paymentflow.com/v1/payments \
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

A payment failure due to a declined card returns an HTTP 402 error. The following is a typical error response:

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
| 400 | `invalid_request_error` | Invalid parameters | Check your request format |
| 401 | `authentication_error` | Invalid API key | Verify your API key |
| 402 | `payment_error` | Payment failed | Display error to customer |
| 404 | `invalid_request_error` | Resource not found | Check the ID in your request |
| 429 | `rate_limit_error` | Too many requests | Implement retry logic with backoff |
| 500 | `api_error` | Server error | Retry the request |

### Best Practices for error handling

1. **Check HTTP status codes** before parsing responses to ensure errors are handled correctly.
2. **Display clear error messages** to customers.
3. **Log detailed error information** for debugging and support.
4. **Implement retry logic** for network failures and server-side (`5xx`) errors.
5. **Do not retry payment errors**. These indicate declined transactions. Ask users to update their payment details.

---

## Step 5: Retrieve a payment

Retrieve payment details using the payment ID.

### Request example

```bash
curl https://sandbox-api.paymentflow.com/v1/payments/pay_1234567890abcdef \
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
  "receipt_url": "https://paymentflow.com/receipts/pay_1234567890abcdef"
}
```

### Using caching

The GET endpoint supports caching to improve performance. The response includes the following headers:

- `Cache-Control` header: Caching directives
- `ETag` header: Resource version identifier

**Conditional request example:**

Use the `If-None-Match` header to return data only if the resource has changed. This reduces unnecessary API requests and bandwidth.

```bash
curl https://sandbox-api.paymentflow.com/v1/payments/pay_1234567890abcdef \
  -H "Authorization: Bearer sk_test_1234567890abcdef" \
  -H 'If-None-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"'
```

If the resource is unchanged, the server returns `304 Not Modified` with no response body.

---

## Step 6: Process a refund

You can refund all or part of a payment. Refunds may take several days to appear on the customer's statement.

### Full refund example

```bash
curl https://sandbox-api.paymentflow.com/v1/refunds \
  -X POST \
  -H "Authorization: Bearer sk_test_1234567890abcdef" \
  -H "Content-Type: application/json" \
  -d '{
    "payment_id": "pay_1234567890abcdef",
    "reason": "requested_by_customer"
  }'
```

### Partial refund example

```bash
curl https://sandbox-api.paymentflow.com/v1/refunds \
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

**Refund reasons:**

Specify a reason when issuing a refund in the request body. Possible values are:

- `duplicate` - The payment was a duplicate charge
- `fraudulent` - The payment was a fraudulent transaction
- `requested_by_customer` - The customer requested a refund.

### Response

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

---

## Step 7: Set up webhooks (optional)

Webhooks notify your application of events such as successful payments or refund updates in real time.

### Why use webhooks?

- **Real-time updates**: Receive notifications immediately when events occur
- **Reliable** - PaymentFlow retries failed webhook deliveries automatically
- **Asynchronous** - Handle long-running processes without blocking API requests

### Register a webhook endpoint

```bash
curl https://sandbox-api.paymentflow.com/v1/webhooks \
  -X POST \
  -H "Authorization: Bearer sk_test_1234567890abcdef" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com/webhooks/paymentflow",
    "enabled_events": [
      "payment.succeeded",
      "payment.failed",
      "refund.created"
    ],
    "description": "Production webhook endpoint"
  }'
```

### Response

```json
{
  "id": "we_1234567890abcdef",
  "object": "webhook_endpoint",
  "url": "https://example.com/webhooks/paymentflow",
  "enabled_events": [
    "payment.succeeded",
    "payment.failed",
    "refund.created"
  ],
  "status": "enabled",
  "description": "Production webhook endpoint",
  "secret": "{secret}",
  "created": 1699564800
}
```

Save the `secret` to verify incoming webhook signatures and confirm requests are from PaymentFlow. Register your webhook endpoint and test event delivery to ensure your setup is correct.

### Webhook payload example
To test your integration, make a payment to trigger a payment event. Check your webhook URL for the POST request with event details. Review the payload to confirm correct event delivery.

```json
{
  "id": "evt_1234567890abcdef",
  "type": "payment.succeeded",
  "created": 1699564800,
  "data": {
    "object": {
      "id": "pay_1234567890abcdef",
      "amount": 2999,
      "currency": "usd",
      "status": "succeeded",
      "customer": {
        "email": "customer@example.com",
        "name": "Jane Doe"
      }
    }
  }
}
```

### Verify webhook signatures

Verify the webhook signature on each incoming request to confirm it is from PaymentFlow. This protects your application from unauthorized events.

**Node.js Example:**

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(payload, signature, secret) {
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(payload)
    .digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}

// In your webhook handler
app.post('/webhooks/paymentflow', (req, res) => {
  const signature = req.headers['paymentflow-signature'];
  const payload = JSON.stringify(req.body);
  
  if (!verifyWebhookSignature(payload, signature, process.env.WEBHOOK_SECRET)) {
    return res.status(401).send('Invalid signature');
  }
  
  // Process the webhook event
  const event = req.body;
  console.log('Received event:', event.type);
  
  res.status(200).send('Webhook received');
});
```

---

## Understanding rate limits

PaymentFlow uses rate limits to maintain API stability and ensure fair usage.

### Rate limit tiers

| Tier | Requests per Minute | Use Case |
|------|---------------------|----------|
| Standard | 100 | Most applications |
| Premium | 1,000 | High-volume merchants |

### Rate limit headers

Each API response includes your current rate limit details.

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1699564860
```

- `X-RateLimit-Limit`: Maximum requests allowed per minute.
- `X-RateLimit-Remaining`: Requests remaining in current window.
- `X-RateLimit-Reset`: Unix timestamp when the limit resets.

### Handling rate limits

If you go over the rate limit, the API returns a `429 Too Many Requests` response.

```json
{
  "error": {
    "type": "rate_limit_error",
    "code": "rate_limit_exceeded",
    "message": "You have exceeded your rate limit. Please retry after 60 seconds."
  }
}
```

**Best practices for managing rate limits:**

1. **Monitor rate limit headers** in each response.
2. **Use exponential backoff** for retries.
3. **Cache GET requests** where possible, such as for payments or webhooks lists.
4. **Batch operations** when applicable.
5. **Request a tier upgrade** if you consistently reach the rate limit.

**Retry logic (pseudocode) example:**

The following pseudocode demonstrates how to retry a failed operation using exponential backoff. Use this pattern to handle temporary errors, such as rate limiting, by waiting and retrying.

```javascript
async function makeRequestWithRetry(url, options, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    const response = await fetch(url, options);
    
    if (response.status === 429) {
      const retryAfter = response.headers.get('Retry-After') || 60;
      const delay = Math.pow(2, attempt) * 1000; // Exponential backoff
      await sleep(Math.max(delay, retryAfter * 1000));
      continue;
    }
    
    return response;
  }
  
  throw new Error('Max retries exceeded');
}
```

---

## Idempotency

Idempotency ensures you do not incur duplicate charges when retrying a request.

### When to use idempotency

Use idempotent requests in the following situations:

- Payment creation (`POST /payments`)
- Refund creation (`POST /refunds`)
- Any operation where duplicate actions could cause issues

### How to use idempotency keys

Add an `Idempotency-Key` header with a unique value, such as a UUID:

```bash
curl https://sandbox-api.paymentflow.com/v1/payments \
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

**How idempotency works:**

1. The first request with a specific idempotency key is processed normally.
2. Subsequent requests with the same key return the cached response.
3. Idempotency keys expire after 24 hours.

**Implementation example:**

```javascript
const { v4: uuidv4 } = require('uuid');

async function createPayment(paymentData) {
  const idempotencyKey = uuidv4();
  
  const response = await fetch('https://sandbox-api.paymentflow.com/v1/payments', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.PAYMENTFLOW_API_KEY}`,
      'Content-Type': 'application/json',
      'Idempotency-Key': idempotencyKey
    },
    body: JSON.stringify(paymentData)
  });
  
  return response.json();
}
```

---

## Next steps

You have completed the following steps:

✅ Set up your PaymentFlow account and API keys  
✅ Processed your first test payment  
✅ Handled errors gracefully  
✅ Retrieved payment details  
✅ Created a refund  
✅ (Optional) Set up webhooks for real-time notifications

### Going to production

To process live payments, complete the following steps:

1. **Complete your account verification** in the Dashboard
2. **Switch to live API keys** (prefix: `sk_live_`)
3. **Update your base URL** to `https://api.paymentflow.com/v1`
4. **Review security practices** - Ensure keys are stored securely
5. **Set up monitoring** - Track payment success rates and errors
6. **Configure webhooks** for production URLs
7. **Test with small amounts** before processing larger transactions

---

## Quick reference

### Essential cURL commands

**Create a payment:**
```bash
curl https://sandbox-api.paymentflow.com/v1/payments \
  -X POST \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"amount": 2999, "currency": "usd", "payment_method": {...}}'
```

**Retrieve a payment:**
```bash
curl https://sandbox-api.paymentflow.com/v1/payments/PAYMENT_ID \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Create a refund:**
```bash
curl https://sandbox-api.paymentflow.com/v1/refunds \
  -X POST \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"payment_id": "PAYMENT_ID", "reason": "requested_by_customer"}'
```

**List webhooks:**
```bash
curl https://sandbox-api.paymentflow.com/v1/webhooks \
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

---

**Last updated:** December 2024 | **API Version:** 1.0.0
