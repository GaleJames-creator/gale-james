## Webhooks

When an event occurs in the service, it triggers a webhook and sends the event details to your chosen endpoint URL so your system receives real-time updates. Your app can immediately respond to events such as order updates, subscription changes, or refunds. This flow keeps your systems in sync and allows developers to automate workflows, improve user experiences, and keep data current without inefficient polling.

The service notifies your endpoint URL of API events via webhooks, each carrying the event type and timestamp. Manage all webhooks through the Admin API's Webhooks Service.

## Tutorials

- [Creating a webhook](../tutorials/webhook-creating-your-first.md)

## How-to-guides

- [How to manage security for webhook and data transmission](../how-to-guides/webhooks-data-transmission.md)
- [How to manage webhooks using the Admin UI](../how-to-guides/webhooks-Admin-UI.md)
- [How to retrieve an anonymous shopper's order and product information](../how-to-guides/webhooks-anonymous-shopper.md)

## Reference

- [Subscription webhook examples](../reference/webhook-subscription-examples.md)
- [Webhook definitions](../reference/webhook-definitions.md)
