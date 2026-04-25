# Payment API error codes

This overview explains common API error codes and directs you to the appropriate guide for your situation. For detailed implementation guidance, see the related guides below.

## Error types

Use the following table to identify your error code and find the appropriate resolution path.

| HTTP Status | Error Type | Description | Action |
| ----------- | --------- | ----------- | ------- |
| `400` | `invalid_request_error` | Invalid parameters | Validate required fields, data types, and formatting, then resubmit. |
| `401` | `authentication_error` | Invalid API key | Verify your API key. |
| `402` | `payment_error` | Payment failed | Notify the customer that their payment failed and ask them to update their payment method. For a complete list of decline codes, see [Payment decline codes](./payment-decline-codes.md). |
| `403` | `authorization_error` | Insufficient permissions | Check your API key permissions and access rights. |
| `404` | `not_found_error` | Resource not found | Check the ID in your request. |
| `429` | `rate_limit_error` | Too many requests | Implement retry logic with backoff. |
| `5xx` | `api_error` | Server error | Implement retry logic with backoff. |

## Related guides

Based on your situation, refer to the appropriate guide:

* **Authorization failures** — [Handling authorization failures](../how-to-guides/handling-authorization-failures.md)
* **Retry logic** — [Implementing retry logic for failed payments](../how-to-guides/implementing-retry-logic.md)
* **Rate limits** — [Managing API rate limits](../how-to-guides/managing-api-rate-limits.md)
* **Paymend decline codes - [Payment decline codes](./payment-decline-codes.md)

---

Last updated: April 2026
