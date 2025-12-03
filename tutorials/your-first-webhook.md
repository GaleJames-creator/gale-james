# Your first webhook
---
Webhooks send a POST request to your URL when an event occurs, such as a payment or refund. This keeps your system updated without the need for polling.

## Why use webhooks?

Webhooks notify you about events as soon as they happen, automatically retry if delivery fails, and let you handle tasks in the background without blocking your main processes.

## Register a webhook endpoint

To receive real-time event notifications, register a webhook endpoint. Provide a publicly accessible URL, such as `https://example.com/webhooks/paymentflow`. The service sends event data to this URL when an event occurs. Use HTTPS to encrypt data in transit. Protect your endpoint with authentication to prevent unauthorized access. Registering your endpoint allows your app to update immediately when a payment or refund happens.

In the following example, an event is triggered when a payment succeeds (`payment.succeeded`), a payment fails (`payment.failed`), or a refund is issued (`refund.created`).

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

The 'secret' is a unique key used to sign webhook payloads and verify that data comes from PaymentFlow. The signature is included in the 'PaymentFlow-Signature' header. Store secrets securely, such as in environment variables or a secrets manager. Rotate secrets regularly and revoke old ones to keep your integration secure.

### Webhook payload example

Test your webhook by making a payment and checking the POST request data at your endpoint. For local testing, use a tool like ngrok to expose your server. Use the PaymentFlow sandbox to safely simulate different scenarios before deploying.

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

Always verify webhook signatures to confirm requests are from PaymentFlow. If verification fails, log the attempt, return a 401 status, and set up alerts for repeated failures. You can temporarily disable the endpoint if needed to maintain security.

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
  
  // Implement idempotency to ensure duplicated events don't cause repeated processing
  const eventId = event.id;
  if (isEventAlreadyProcessed(eventId)) {
    return res.status(200).send('Event already processed');
  }

  // Mark the event as processed

  markEventAsProcessed(eventId);

  res.status(200).send('Webhook received');
});
```
