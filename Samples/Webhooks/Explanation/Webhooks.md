## Webhooks

When an event occurs in the service, it triggers a webhook and sends the event details to your chosen URL so your system receives real-time updates. Your app can immediately respond to events such as order updates, subscription changes, or refunds. This flow keeps your systems in sync and allows developers to automate workflows, improve user experiences, and keep data current without inefficient polling.

The service notifies your endpoint URL of API events via webhooks, each carrying the event type and timestamp. Manage all webhooks through the Admin API's Webhooks Service.

Creating a webhook allows your system to receive real-time notifications from the service, enabling you to automate and enhance your processes. Whether you're tracking subscription updates, inventory statuses, or other events, setting up a webhook is straightforward: open your firewall to the service's IPs, create the webhook in your service project, and configure your system to respond to the events. See [Creating a webhook](../How-to-guides/How-to-manage-webhooks-using-the-Admin-UI.md#creating-a-webhook.md) and [Subscription webhook examples](../Reference/Subscription-webhook-examples.md) for additional information.
