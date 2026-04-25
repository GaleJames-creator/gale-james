# Preventing duplicate charges with the `Idempotency-Key`

Duplicate charges frustrate customers and create support issues. The `Idempotency-Key`ensures each payment is processed only once, even if the same request is sent multiple times due to network issues or retries. This guide explains how the `Idempotency-Key` works, why it matters, and how to use it effectively in your payment system.

> For a complete explanation of the `Idempotency-Key` and how it wors, see [Understanding the `Idempotency-Key`](../explanation/understanding-the-idempotency-key.md).

## Step 1. Generate the `Idempotency-Key`

Always use the `Idempotency-Key` when creating, updateing, or deleting a payment.

```python
import uuid

idempotency_key = str(uuid.uuid4())

response = requests.post(
    f"{BASE_URL}/payments",
    headers={
        'Authorization': f'Bearer {token}',
        'Idempotency-Key': idempotency_key
    },
    json=payment_data
)
```

## Step 2. Add idempotency to prevent duplicate charges

Always use the same `idempotency_key` for each retry of the same payment. This prevents duplicate charges if the payment request is processed multiple times.

```python
headers={
    "Authorization": f"Bearer {api_key}",  # Never log API keys.
    "Idempotency-Key": payment_data.get("idempotency_key", "")
}
```

> **Note**: Load the api_key from an environmental variable:

```python
import os
api_key = os.getenv("PAYMENT_API_KEY")
```

> **Note**: Load the api_key from an environmental variable:

```python
import os
api_key = os.getenv("PAYMENT_API_KEY")
```

## Next steps

### Explanation

* [Understanding the `Idempotency-Key`](../explanation/understanding-the-idempotency-key.md): Covers the idempotency concepts, how they work, and when to use them.

## Reference

* [`Idempotency-Key`](../reference/Idempotency-Key): Covers essential idempotency information.

---

Last updated: April 2026
