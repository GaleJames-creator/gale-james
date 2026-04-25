# Webhook definitions

Webhooks allow one system to send real-time data to another whenever a specific event happens. By delivering event notifications to a specified URL, your application can react automatically to changes or updates.

## Webhook ID

A Webhook ID uniquely identifies a webhook, letting you find, edit, enable, disable, or delete it in the Admin UI.

## Endpoint URL

An endpoint URL is where webhooks send events. It must use `http://` or `https://`, and your server should reply with an HTTP `2xx` status code to acknowledge the request. If your endpoint fails to return a `2xx` status code for several consecutive attempts, it may be disabled. See[Managing webhooks using the Admin UI](../how-to-guides/managing-webhooks-using-Admin-UI.md) for details on re-enabling a disabled endpoint.

## Secret

A secret is a unique key that the service uses to sign webhook payloads sent to your endpoint URL. This cryptographic signature guarantees data integrity and confirms authenticity. You can reveal and rotate a webhook's secret in the Admin UI. Incorporate this secret into your endpoint's verification process to validate received payloads.

---

## Next steps

## Explanation

* [Understanding webhooks](../explanation/understanding-webhooks.md): Covers the key webhook concepts, how they work, and when to use them.

### How-to guides

* [Managing webhooks using the Admin API](../how-to-guides/managing-webhooks-using-Admin-UI.md): How to create, edit, enable, disable, or delete webhooks.
* [Managing security for webhook data transmission](../how-to-guides/managing-security-webhook-data-transmission.md): How to protect webhook data

### Reference

* [Webhook subscription examples](./webhook-subscription-examples.md): Covers two subscription webhook examples: 201 Created webhook and full subscription payload.

Last updated: April 2026
