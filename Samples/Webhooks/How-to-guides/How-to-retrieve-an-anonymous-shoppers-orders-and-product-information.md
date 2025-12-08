# How to retrieve an anonymous shopper's orders and product information

This guide assumes you received a webhook containg an anonymous shopper's user ID (`userId). To retrieve an anonymous shopper's orders and product information, follow these steps:

## Step 1: Obtain a token for the anonymous shopper

Begin by obtaining an access token using the anonymous shopper's ID by sending a POST request to the services's OAuth token endpoint:

```json
  POST https://api.service.com/oauth20/token?grant_type=client_credentials&dr_userid={anonymousShopperId}
```

Replace `{anonymousShopperId}` with the actual shopper ID provided in the subscription details under `shopper.id`.

## Step 2: Use the token to access orders and product Information

Once you have received a session token, you can access the shopper's order and product information.

1. **Get product information**: Retrieve product details by sendin a GET /products request. Replace `{productId}` with the specific product ID you wish to view.

```json
  GET https://api.service.com/v1/shoppers/me/products/{productId}?expand=all
```

2. **Get order information**: Fetch order details by making a GET /orders request. This allows you to list all orders associated with the anonymous shopper.

```json
  GET https://api.service.com/v1/shoppers/me/orders?expand=all
```
