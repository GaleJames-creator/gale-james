# Understanding webhooks

Webhooks deliver real-time event notifications from one service to another, triggering actions in your app without polling. You can use them to update apps, send notifications, or sync data.

## Key concepts

* **Event types**: Each webhook notification includes an event type (such as payment_succeeded, subscription_created, or order_updated). This tells your app what action occurred so you can respond appropriately.
* **Endpoint URLs**: You set up an endpoint URL in your app to receive webhook POST requests. This is where the external service delivers event data in real time.
* **Payload format**: Webhooks typically send data in JSON format, containing information about the event and all relevant objects (such as customer or order info).
* **Security**: To protect your webhook endpoint, use secret tokens or signatures to verify that requests are from trusted sources. Validate incoming requests and avoid exposing sensitive data in URLs.
* **Reliability**: Webhook services frequently retry notifications if your endpoint is temporarily unavailable. Ensure your endpoint can handle repeated notifications and return the correct HTTP status code.
* **Idempotency**: Your webhook handling logic should be idempotent, meaning processing the same event more than once has the same effect as processing it once. This prevents duplicate actions if an event is retried.
* **Logging and monitoring**: Track webhook deliveries, failures, and processing results to help with troubleshooting and maintenance.
* **Scalability**: Design your webhook handler to process events efficiently, even under high load, so your app stays responsive and reliable.

## Why use webhooks?

* Receive event notifications as soon as they happen.
* Automatically retry notifications if delivery fails.
* Handle event notifications in the background without blocking your main processes.

## How it works

The event triggers a webhook and sends the event details to your chosen endpoint URL. Your app can immediately respond to events such as order updates, subscription changes, or refunds. This flow keeps your system in sync, automates workflows, improves user experiences, and keeps data current without inefficient polling.

## Next steps

### How-to guides

* [Retrieving anonymous customer information](../how-to-guides/retrieving-anonymous-customer-information.md): How to retrieve an anonymous customer's order and product information.
* [Manage security for webhook data transmission](../how-to-guides/managing-security-webhook-data-transmission.md): How to protect webhook data.

---

Last updated:  April 2026
