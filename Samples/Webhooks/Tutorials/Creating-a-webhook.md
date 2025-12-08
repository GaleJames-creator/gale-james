# Creating a webhook

Webhooks send a POST request to your URL when an event occurs, such as a payment or refund. This keeps your system updated without polling.

## Why use webhooks?

Webhooks notify you about events as soon as they happen, automatically retry if delivery fails, and let you handle tasks in the background without blocking your main processes.

## What you'll learn

* [How to register a webhook endpoint](#step-1-register-a-webhook-endpoint)
* [How to verify a webhook signature](#step-2-verify-webhook-signatures)

## Prerequisites

Before getting started, ensure you have:

- **A public, secure endpoint** where you want to receive webhook events that can process HTTPS POST requests from the service sending the webhook (e.g., GitHub, Stripe, etc.).
  
  **Best Practice**: Use security measures like secret tokens or signature verification to make sure requests are really from the expected service.
  
- **A testing or staging environment** to validate your webhook endpoint before going live.

## Step 1: Register a webhook endpoint

To receive real-time event notifications, register a webhook endpoint. Provide a publicly accessible URL, such as `https://www.acme.com/webhooks/payment`. The service sends event data to this URL when an event occurs. Use HTTPS to encrypt data in transit. Protect your endpoint with authentication to prevent unauthorized access. Registering your endpoint lets your app update immediately when a payment or refund occurs.

In the following example, an event is triggered when a payment succeeds (`payment.succeeded`), a payment fails (`payment.failed`), or a refund is issued (`refund.created`).

```bash
curl https://sandbox-api.payment.com/v1/webhooks \
  -X POST \
  -H "Authorization: Bearer sk_test_1234567890abcdef" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://example.com/webhooks/payment",
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
  "url": "https://example.com/webhooks/payment",
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

The `secret` is a unique key used to sign webhook payloads and verify that data comes from the Payment API. The signature is included in the 'Payment-Signature' header. Store secrets securely, such as in environment variables or a secrets manager. Rotate secrets regularly and revoke old ones to keep your integration secure.

### Webhook payload example

Test your webhook by making a payment and checking the POST request data at your endpoint. For local testing, use a tool like ngrok to expose your server. Use the Payment sandbox to safely simulate different scenarios before deploying.

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

### Step 2: Verify webhook signatures

Always verify webhook signatures to confirm requests are from Payment. If verification fails, log the attempt, return a 401 status, and set up alerts for repeated failures. You can temporarily disable the endpoint if needed to maintain security.

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
app.post('/webhooks/payment', (req, res) => {
  const signature = req.headers['payment-signature'];
  const payload = JSON.stringify(req.body);
  
  if (!verifyWebhookSignature(payload, signature, process.env.WEBHOOK_SECRET)) {
    return res.status(401).send('Invalid signature');
  }
  
  // Process the webhook event
  const event = req.body;
  console.log('Received event:', event.type);
  
  // Implement idempotency to prevent duplicate processing of repeated events.
  const eventId = event.id;
  if (isEventAlreadyProcessed(eventId)) {
    return res.status(200).send('Event already processed');
  }

  // Mark the event as processed

  markEventAsProcessed(eventId);

  res.status(200).send('Webhook received');
});
```

## What you've accomplished

You have completed the following steps:

✅ Registered a webhook endpoint<br>
✅ Verified webhook signatures

### Next steps

Ready for more? Check out:

- [How to manage security for webhook data transmission](../How-to-guides/How-to-manage-security-for-webhook-data-transmission.md)
- [How to manage webhooks using the Admin UI](..//How-to-guides/How-to-manage-webhooks-using-the-Admin-UI.md)
- [How to retrieve an anonymous shopper's orders and product information](../How-to-retrieve-an-anonymous-shoppers-orders-and-product-information.md)




