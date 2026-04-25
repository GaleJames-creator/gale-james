# Creating a webhook

Webhooks send a POST request to your URL when an event occurs, such as a payment or refund. This keeps your system updated without polling.

## What you'll learn

* [How to register a webhook endpoint](#step-1-register-a-webhook-endpoint)
* [How to verify a webhook signature](#step-2-verify-webhook-signatures)

## Prerequisites

Before getting started, ensure you have:

* **A public, secure endpoint** where you want to receive webhook events that can process HTTPS POST requests from the payment endpoint sending the webhook (e.g., GitHub, Stripe, etc.).
  
  > **Tip**: Use security measures such as secret tokens or signature verification to ensure requests are really from the expected payment endpoint.
  
* **A testing or staging environment** to validate your webhook endpoint before going live.

## Step 1: Register a webhook endpoint

To receive real-time event notifications, register a webhook endpoint and provide a publicly accessible URL, such as `https://example.com/webhooks/payment`. The payment endpoint sends event data to this URL when an event occurs.

> **Note**: Use HTTPS to encrypt data in transit. Protect your endpoint with authentication to prevent unauthorized access.

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

The `secret` is a unique key used to sign webhook payloads and verify that data comes from the Payment endpoint. The signature is included in the `Payment-Signature` header. Store secrets securely, such as in environment variables or a secrets manager. Rotate secrets regularly and revoke old ones to keep your integration secure.

### Webhook payload

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

## Step 2: Verify webhook signatures

Always verify webhook signatures to confirm requests are from Payment. If verification fails, log the attempt, return a 401 status, and set up alerts for repeated failures. You can temporarily disable the endpoint if needed to maintain security.

### Verify a signature in Node.js

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
  // Use raw body string for signature verification, not parsed JSON
  // Configure Express to preserve raw body: app.use(express.raw({type: 'application/json'}))
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

### How-to guides

* [Managing security for webhook data transmission](../how-to-guides/managing-security-webhook-data-transmission.md): How to protect webhook data.
* [Managing webhooks using the Admin UI](../how-to-guides/managing-webhooks-using-Admin-UI.md): How to create, edit, enable, disable, or delete webhooks, and manage secrets.

---

Last updated: April 2026
