# Retrieve an anonymous shopper's orders and product information

This guide shows how to retrieve an anonymous shopper's orders and product information using their user ID (userId).

## Prerequisites

- You have received a webhook containing the anonymous shopper's user ID (`userId`).

- You have access to the service's API endpoints and credentials as required.

## Step 1: Obtain an access token for the anonymous shopper

Send a `POST` request to the service's OAuth token endpoint to obtain an access token for the anonymous shopper. Use the following format:

```json
  POST https://api.service.com/oauth20/token?grant_type=client_credentials&dr_userid={anonymousShopperId}
```

Replace `{anonymousShopperId}` with the actual user ID from your webhook payload (typically found in `shopper.id`).

## Step 2: Use the access token to retrieve orders and product information

After obtaining the session token, include it in the `Authorization` header as a Bearer token for all subsequent requests to access the anonymous shopper's data.

1. **To retrieve product information**: Send a `GET` request to the `/products` endpoint, specifying the desired product ID. As shown in the following example:

```json
  GET https://api.service.com/v1/shoppers/me/products/{productId}?expand=all
```

2. **To list all orders for the anonymous shopper**: Send a `GET` request to the `/orders` endpoint. As shown in the following example:

```json
  GET https://api.service.com/v1/shoppers/me/orders?expand=all
```
