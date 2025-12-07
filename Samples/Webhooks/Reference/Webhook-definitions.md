# Webhook definitions

## Webhook ID

A Webhook ID is a unique identifier assigned to a specific webhook. This ID is crucial for managing webhooks, allowing precise control over operations such as search, edit, enable, disable, and delete. If you need to locate a particular webhook within the service’s Admin UI, knowing its Webhook ID will allow you to search for it directly on the Webhook Service page.

## Endpoint URL

An endpoint URL is the destination on your server where the service delivers webhook events. This URL must begin with http:// or https://, and your service must return a `2xx` HTTP status code to acknowledge successful receipt of each event. If your endpoint fails to acknowledge events with a `2xx` status code over several days, the service may disable it. Properly configure and maintain your endpoint to ensure uninterrupted receipt and processing of webhook events.

## Secret

A secret is a unique key that the service uses to sign webhook payloads sent to your endpoint URL. This cryptographic signature guarantees the integrity and authenticity of the data, confirming it has not been altered in transit. You can view or update a webhook's secret in your account settings. Incorporate this secret into your endpoint's verification process to validate the source and integrity of received payloads.

You can reveal and rotate a webhook's secret in Admin UI.
