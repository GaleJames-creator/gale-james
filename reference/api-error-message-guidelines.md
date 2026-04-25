# API error message guidelines

Make API error messages clear and actionable so users understand the problem and how to fix it. A clear and actionable error message:

* **Directly states the problem**: It tells the user what went wrong in simple, specific terms.
* **Avoids jargon**: Uses words that any user can understand.
* **Offers a solution**: Explains what the user should do next to fix the issue.
* **Is concise**: Gets to the point without unnecessary details.

## Bad (exposes technical details)

```bash
"Payment failed with code: insufficient_funds. Error ID: 402-0001"
```

## Good (actionable and clear)

```bash
"We couldn't process your payment. Please update your payment method or contact your bank for assistance."
```

## Next steps

### Explanation

* [Understanding authorization and capture](../explanation/understanding-authorization-capture.md): Explains the two-step payment process, authorization holds, and decline types.

### How-to guides

* [Processing a refund](../how-to-guides/processing-a-refund.md): How to process a refund.

### Reference

* [Payment API error codes](./payment-api-error-codes.md): Reference for common API error codes and resolution paths.
* [API reference guide](./api-reference-guide.md): Covers three API endpoints: retrieve available payment options, update payment options, and create an order refund.
* [Payment decline codes](./payment-decline-codes.md): Complete reference of all authorization decline codes with hard/soft classification.

---

Last updated: April 2026
