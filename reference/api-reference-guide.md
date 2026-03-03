# API reference guide

This guide provides details on two API endpoints.

* [Retrieve available payment methods](#retrieve-available-payment-methods)
* [Create an order refund](#create-an-order-refund)

## Retrieve available payment methods
Get a list of the available payment options.

### Endpoint
```http
GET /v1/payment-options
```

### Request example

```bash
curl -X GET "https://api.sandbox-api.com/v1/payment-options" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
```

### Response (200 OK)

```json
{
  "paymentOptions": {
    "uri": "https://api.sandbox-api.com/v1/shoppers/me/payment-options",
    "paymentOption": [
      {
        "sourceId": "a231f38d-3a07-4a13-96ed-89693ba7d56c",
        "uri": "https://api.sandbox-api.com/v1/shoppers/me/payment-options/740865108",
        "id": 15699113789,
        "nickName": "Default",
        "isDefault": "true"
      }
    ]
  }
}
```

### Response (400 Bad Request)

The parameter is invalid. Enter a valid parameter and try again.

```json
{
  "code": "invalid_parameter",
  "description": "The parameter is invalid. Enter a valid parameter and try again."
}
```

### Response (401 Unauthorized)

The token is invalid. Enter a valid token and try again.

```json
{
  "code": "invalid_token",
  "description": "The token is invalid. Enter a valid token and try again."
}
```

### Response (409 Conflict)

The possible errors associated with this conflict response are:

- `resource-already-exists`: A payment Option with this nickname already exists. Provide a unique nickname for the payment option and try again.
- `invalid-payment-source`: The `paymentSource` with source ID (`sourceId`) is not valid. Provide a valid `paymentSource` for the `sourceId` and try again.
- `resource-already-exists`: The `paymentSource` with source ID (`sourceId`) already exists. Provide a new `paymentSource` for the `sourceId` and try again.
- `invalid-state-code`: The state code is incorrect. Enter a valid 2-digit state code and try again.
- `invalid-postal-code`: The postal code is incorrect. Enter a valid postal code in the format `12345` or `12345-6789` and try again.

```json
{
  "code": "resource-already-exists",
  "description": "A Payment Option with a nickname of [nickname] already exists."
}
```

## Create an order refund

Use the refunds API to send an order refund to the customer.

### Endpoint

```http
POST /v1/orders/{orderId}/refunds
```

### Path parameters

- `orderId` (required): Unique identifier of the order. In this example, the `orderId` is `12345`.

### Request example

```bash
curl https://sandbox-api.com/v1/orders/12345/refunds \
  -X POST \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "Order Level",
    "category": "PRODUCT_LEVEL_PRODUCT",
    "reason": "REQUESTED_BY_CUSTOMER",
    "comments": "The customer said the product was damaged.",
    "refundAmount": {
        "value": 100.00
    }
  }
'
```

### Request parameters

In the request body, include the following required fields:

| Required parameters | Description                                                                                                                 | 
|---------------------|-----------------------------------------------------------------------------------------------------------------------------|
| `type`              | The type of refund, such as `Order Level` or `Product Level` refund.                                                        |
| `category`          | The enum for the category. Possible values: `PRODUCT_LEVEL_PRODUCT`,  `PRODUCT_LEVEL_SHIPPING`, `PRODUCT_LEVEL_FEE`,`PRODUCT_LEVEL_DUTIES_AND_TARIFFS`, `ORDER_LEVEL_PRODUCT`, `ORDER_LEVEL_SHIPPING`, `ORDER_LEVEL_FEE`, `ORDER_LEVEL_FULL`, `ORDER_LEVEL_TAX_EXEMPT`, `ORDER_LEVEL_DUTIES_AND_TARIFFS`   |
| `reason`            | The enum for the reason. Possible values: `CUSTOMER_SATISFACTION_ISSUE`, `DAMAGED_PRODUCT`, `DELAYED_SHIPPING`, `FEE_CHARGED_INCORRECTLY`,`FEE_EXEMPT_CUSTOMER`, `FRAUD`, `MATCH_PROMOTIONAL_PRICE`, `MISSING_ITEMS_FROM_ORDER`, `NEVER_RECEIVED`, `ORDER_PROCESSING_ERROR`, `PHONE_ORDER_ERROR`, `PRODUCT_SHOULD_NOT_HAVE_FEE`, `TAX_EXEMPT`, `UNABLE_TO_SHIP_TO_COUNTRY`, `UNDELIVERABLE_ADDRESS`, `VENDOR_APPROVED_REFUND`                                |
| `comments`          | Any comments for the refund (e.g., `The customer said the product was damaged.`).                                           |
| `value`             | The value for the `refundAmount`.                                                                                           |

### Response (200 OK)

```json
{
  "id": "ref_abc123",
  "status": "pending",
  "type": "Order Level",
  "refundAmount": {
    "value": 100.00,
    "currency": "USD"
  },
  "createdAt": "2026-01-02T14:30:00Z"
}
```

### Response (400 Bad Request)

The parameter is invalid. Enter a valid parameter and try again.

```json
{
  "code": "invalid_parameter",
  "description": "The parameter is invalid. Enter a valid parameter and try again."
}
```

### Response (403 Forbidden)

The token is invalid. Provide the correct token and try again.

### Response (404 Not Found)

The order ID or line item was not found, or the refund is invalid. Provide the correct order ID, line item, or refund, and try again.

```json
{
  "errors": {
    "code": "resource-not-found",
    "subcode": "invalid-order-id",
    "description": "The order ID is invalid. Provide the correct order ID and try again.",
    "message": "Invalid order ID."
  }
}
```

### Response (409 Conflict)

The refund type, category, reason, or refund amount is incorrect. Provide the correct value and try again.

```json
{
  "errors": {
    "code": "invalid-refund-request",
    "subcode": "invalid-refund-type",
    "description": "The refund type is invalid. Enter the correct refund type and try again.",
    "message": "Invalid refund type."
  }
}
```
