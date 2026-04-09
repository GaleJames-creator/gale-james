# API reference guide

This guide provides details on three API endpoints.

* [Retrieve available payment options](#retrieve-available-payment-options)
* [Update the customer's payment options](#update-the-customers-payment-options)
* [Create an order refund](#create-an-order-refund)

## Retrieve available payment options

Get a list of the available payment options.

### Payment options endpoint

```http
GET /v1/shoppers/me/payment-options
```

### Get payment options request example

```bash
curl -X GET "https://api.sandbox-api.com/v1/shoppers/me/payment-options" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" 
```

### 200 OK response for getting payment options

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

### 400 Bad Request response for getting payment options

The parameter is invalid. Enter a valid parameter and try again.

```json
{
  "code": "invalid_parameter",
  "description": "The parameter is invalid. Enter a valid parameter and try again."
}
```

### 401 Unauthorized response for getting payment options

The token is invalid. Enter a valid token and try again.

```json
{
  "code": "invalid_token",
  "description": "The token is invalid. Enter a valid token and try again."
}
```

## Update the customer's payment options

You can update the customer's payment options.

### Update payment option endpoint

```http
POST /v1/shoppers/me/payment-options/{paymentOptionId}
```

### Update payment option path parameters

* `paymentOptionId` (required): Unique identifier of the payment option (e.g., `123456789`).

### Update payment option request example

```bash
curl https://api.sandbox-api.com/v1/shoppers/me/payment-options/123456789 \
  -X POST \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "paymentOption": {
      "nickName": "DRBank",
      "isDefault": "true",
      "sourceId": "c25fcccc-6e57-4d93-bafb-a6765510a422"
    },
    "shopper": {
      "ipAddress": "192.168.100.102"
    }
  }
'
```

### Update payment option request parameters

In the request body, include the following optional fields:

| Parameter | Location | Type | Required	| Description |
| ------------------------------- | ------------- | --------- | --------- | -------------------------------------------------------------------------------------------- |  
| `paymentOptionId` | path | string | Yes | The unique identifier for the payment option (e.g., `123456789`) | 
| `sourceId` | body (JSON) | string | No | The unique identifier for the payment source (e.g., `c25fcccc-6e57-4d93-bafb-a6765510a422`) |
| `paymentOption.nickName` | body (JSON) | string | No | Name of payment option (e.g., `WFBank`) | 
| `paymentOption.isDefault` | body (JSON) | string | No | Determines if the payment option is the customer's default payment option (e.g., `true`) | 
| `shopper.ipAddress` | body (JSON) | string | No | Customer's IP address (e.g., `198.51.100.1`) | 

### 201 Created response updating a payment option

```json
{
  "uri": "https://api.sandbox-api.com/v1/shoppers/me/payment-options/123456789",
  "id": 740865108,
  "nickName": "Default",
  "isDefault": "true",
  "type": "creditCard",
  "sourceId": "a231f38d-3a07-4a13-96ed-89693ba7d56c",
  "sourceClientSecret": "a231f38d-3a07-4a13-96ed-89693ba7d56c_f6d8c951-59c9-4ef3-ac45-9f33c77d2f46",
  "creditCard": {
    "expirationYear": "2030",
    "lastFourDigits": "0000",
    "clientSecret": "a231f38d-3a07-4a13-96ed-89693ba7d56c_f6d8c951-59c9-4ef3-ac45-9f33c77d2f46",
    "expirationMonth": "08",
    "fundingSource": "debit",
    "brand": "Visa",
    "reusable": "true"
  }
}
```

### 204 No Content response for updating a payment option

The server successfully processed the request, but has no content to return in the response body.

### 400 Bad Request response for updating a payment option

The possible errors associated with this bad request response are:

- `invalid-request`: The request was invalid and could not be parsed by the system. Enter valid information and try again.

```json
{
  "errors": {
    "code": "invalid-request",
    "description": "The request could not be parsed. Enter valid information and try again."
  }
}
```

### 401 Unauthorized response for updating a payment option

The token is invalid. Enter a valid token and try again.

```json
{
  "errors": {
    "code": "invalid-token",
    "description": "The token is invalid. Enter a valid token and try again."
  }
}
```

### 409 Conflict response for updating a payment option

The `paymentSource` is not valid. Possible reasons are:

- `invalid-state-code`: The state code is invalid. Provide a valid state code and try again.
- `invalid-postal-code`: The postal code is invalid. Provide a valid postal code and try again.

```json
{
  "code": "invalid-payment-source",
  "description": "A PaymentSource with source ID {sourceId} is not valid for creating a billing option."
}
```

## Create an order refund

Use the refunds API to send an order refund to the customer.

### Refunds endpoint

```http
POST /v1/orders/{orderId}/refunds
```

### Refunds path parameters

- `orderId` (required): Unique identifier of the order. In this example, the `orderId` is `12345`.

### Refund request example

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

### Refund request parameters

In the request body, include the following required fields:

| Required parameters | Description                                                                                                                 | 
|---------------------|-----------------------------------------------------------------------------------------------------------------------------|
| `type`              | The type of refund, such as `Order Level` or `Product Level` refund.                                                        |
| `category`          | The enum for the category. Possible values: `PRODUCT_LEVEL_PRODUCT`,  `PRODUCT_LEVEL_SHIPPING`, `PRODUCT_LEVEL_FEE`,`PRODUCT_LEVEL_DUTIES_AND_TARIFFS`, `ORDER_LEVEL_PRODUCT`, `ORDER_LEVEL_SHIPPING`, `ORDER_LEVEL_FEE`, `ORDER_LEVEL_FULL`, `ORDER_LEVEL_TAX_EXEMPT`, `ORDER_LEVEL_DUTIES_AND_TARIFFS`   |
| `reason`            | The enum for the reason. Possible values: `CUSTOMER_SATISFACTION_ISSUE`, `DAMAGED_PRODUCT`, `DELAYED_SHIPPING`, `FEE_CHARGED_INCORRECTLY`,`FEE_EXEMPT_CUSTOMER`, `FRAUD`, `MATCH_PROMOTIONAL_PRICE`, `MISSING_ITEMS_FROM_ORDER`, `NEVER_RECEIVED`, `ORDER_PROCESSING_ERROR`, `PHONE_ORDER_ERROR`, `PRODUCT_SHOULD_NOT_HAVE_FEE`, `TAX_EXEMPT`, `UNABLE_TO_SHIP_TO_COUNTRY`, `UNDELIVERABLE_ADDRESS`, `VENDOR_APPROVED_REFUND`                                |
| `comments`          | Any comments for the refund (e.g., `The customer said the product was damaged.`).                                           |
| `value`             | The value for the `refundAmount`.                                                                                           |

### 200 OK response for refunds

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

### 400 Bad Request response for refunds

The parameter is invalid. Enter a valid parameter and try again.

```json
{
  "code": "invalid_parameter",
  "description": "The parameter is invalid. Enter a valid parameter and try again."
}
```

### 403 Forbidden response for refunds

The token is invalid. Provide the correct token and try again.

### 404 Not Found response for refunds

The order ID or line item was not found, or the refund is invalid. The possible errors associated with this not-found response are:

- `order-not-found`: The provided `orderId` was not found. Provide the correct `orderId` and try again.
- `invalid-refund`: The `refundAmount`'s `value` or `currency` is invalid. Provide a valid `value` or `currency` for the `refundAmount`, and try again.

```json
{
  "errors": {
    "code": "resource-not-found",
    "subcode": "invalid-order-id",
    "description": "The order ID is invalid.",
    "message": "The order ID is invalid. Correct the order ID and try again."
  }
}

```

### 409 Conflict response for refunds

The refund type, category, reason, or refund amount is incorrect. The possible errors associated with this conflict response are:

- `invalid-refund-type`: The refund `type` is invalid. Provide a valid refund `type` and try again.
- `invalid-refund-category`: The refund `category` is invalid. Provide a valid refund `category` and try again.
- `invalid-refund-reason`: The refund `reason` is invalid. Provide a valid refund `reason` and try again.
- `invalid-refund-amount`: The `refundAmount`'s `value` or `currency` is invalid. Provide a valid `value` or `currency` for the `refundAmount`, and try again.

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
