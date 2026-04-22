# Authorization error message guidelines

Provide a clear explanation of the error and how to resolve it.

## Bad (exposes technical details)

```json
"Payment failed with code: insufficient_funds. Error ID: 402-0001"
```

## Good (actionable and clear)

```json
"We couldn't process your payment. Please update your payment method or contact your bank for assistance."
```

## Next steps

### Explanation

* [Understanding authorization and capture](../explanation/understanding-authorization-capture.md): Explains the two-step payment process, authorization holds, and decline types.

### How to guides

* [Processing a refund](../how-to-guides/processing-a-refund.md): How to process a refund.

### Reference

* [Payment decline codes reference](../reference/decline-codes.md): Complete reference of all authorization decline codes with hard/soft classification.

---

Last updated: April 2026

---

Last updated: April 2026
