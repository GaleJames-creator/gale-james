# Webhook definitions

Webhooks allow one system to send real-time data to another whenever a specific event happens. By delivering event notifications to a specified URL, your application can react automatically to changes or updates.

## Webhook ID

A Webhook ID uniquely identifies a webhook, letting you find, edit, enable, disable, or delete it in the Admin UI.

## Endpoint URL

An endpoint URL is where webhooks send events. It must use `http://` or `https://`, and your server should reply with an HTTP `2xx` status code to acknowledge the request. If too many acknowledgments are missed, the endpoint may be disabled.

## Secret

A secret is a unique key that the service uses to sign webhook payloads sent to your endpoint URL. This cryptographic signature guarantees data integrity and confirms authenticity. You can reveal and rotate a webhook's secret in the Admin UI. Incorporate this secret into your endpoint's verification process to validate received payloads.
