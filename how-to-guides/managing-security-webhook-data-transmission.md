# Managing security for webhook data transmission

Use these security steps to protect webhook data.

## Prerequisites

* An active webhook
* Admin UI access

## Steps

1. Always use HTTPS for endpoint URLs to encrypt transmitted data.
2. Verify each payload using the provided [secret](../reference/webhook-definitions.md#secret) and HMAC-SHA256 signature to ensure authenticity and integrity. See [Revealing a webhook's secret](./managing-webhooks-using-Admin-UI.md#revealing-a-webhooks-secret) for instructions.
3. Configure your endpoint to respond within the platform's timeout window to avoid triggering automatic retries. The platform retries failed deliveries up to 4 times with exponential backoff. See [Managing API rate limits](../reference/payment-api-rate-limits.md) for more information.
4. Only accept incoming requests from the service’s known IP addresses (IP safelisting). This limits webhook events to trusted sources.
5. Log incoming traffic and monitor for anomalies, such as repeated failed verifications or unexpected IP addresses.
6. Rotate secrets every 90 days or immediately after a suspected breach.

## Next steps

### How-to guides

* [Managing webhooks using the Admin UI](./managing-webhooks-using-Admin-UI.md): How to create, edit, enable, disable, or delete webhooks, and manage secrets.

---

Last updated: April 2026
