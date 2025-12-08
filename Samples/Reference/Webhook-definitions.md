# Webhook definitions

## Webhook ID

A Webhook ID is a unique identifier for each webhook. It lets you precisely search, edit, enable, disable, or delete a webhook. Knowing the Webhook ID allows you to look up a webhook directly in the Admin UI.

## Endpoint URL

An endpoint URL is where webhook events are delivered to your server. It must start with http:// or https://, and your service should return an HTTP `2xx` status code to acknowledge each event. If acknowledgments are missed for several days, the endpoint may be disabled. Maintain your endpoint to ensure uninterrupted event processing.

## Secret

A secret is a unique key that the service uses to sign webhook payloads sent to your endpoint URL. This cryptographic signature guarantees the integrity and authenticity of the data, confirming it has not been altered in transit. You can view or update a webhook's secret in your account settings. Incorporate this secret into your endpoint's verification process to validate the source and integrity of received payloads.

You can reveal and rotate a webhook's secret in Admin UI.
