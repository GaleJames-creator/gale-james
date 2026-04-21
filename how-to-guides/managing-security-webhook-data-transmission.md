# Managing security for webhook data transmission

Use these security steps to protect webhook data.

## Prerequisites

* An active webhook
* Admin UI access

## Steps

* Always use HTTPS for endpoint URLs to encrypt transmitted data.
* Verify each payload using the provided [secret](../reference/webhook-definitions.md#secret) to ensure authenticity and integrity. See [Revealing a webhook's secret](./managing-webhooks-using-Admin-UI.md#revealing-a-webhooks-secret) for instructions.
* Limit webhook delivery retries to avoid excessive requests. See [Managing API rate limits](../reference/payment-api-rate-limits.md) for more information.
* Only accept incoming requests from the service’s known IP addresses (IP safelisting). This limits webhook events to trusted sources.
* Log incoming traffic and monitor for anomalies or failed verifications.
* Regularly review and rotate secrets to reduce the risk of unauthorized access.

## Next steps

### How-to guides

* [Managing webhooks using the Admin UI](./managing-webhooks-using-Admin-UI.md)

---

Last updated: April 2026
