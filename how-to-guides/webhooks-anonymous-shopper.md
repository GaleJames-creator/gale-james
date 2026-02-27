# Retrieve an anonymous customer's order and product information

This guide shows you how to retrieve an anonymous customer's order and product information using their user ID (`userId`).

## Prerequisites

- You have received a webhook containing the anonymous customer's user ID (`userId`).

- You have access to the service's API endpoints and credentials as required.

## Step 1: Obtain an access token for the anonymous customer

Send a `POST` request to the service's OAuth token endpoint to obtain an access token for the anonymous customer. Use the following format:

```http
  POST https://api.service.com/oauth20/token?grant_type=client_credentials&dr_userid={anonymousShopperId}
```

Replace `{anonymousShopperId}` with the actual customer's user ID from your webhook payload (typically found in `shopper.id`).

## Step 2: Use the access token to retrieve orders and product information

After obtaining the session token, include it in the `Authorization` header as a Bearer token for all subsequent requests to access the anonymous customer's data.

### To retrieve product information

Send a `GET` request to the `/products` endpoint, specifying the product ID (`productId`). 

```http
  GET https://api.service.com/v1/shoppers/me/products/{productId}
```

A successful response returns a 200 OK status code.

```json
{
  "product": {
    "id": 64578500,
    "name": "Dungeon Crawl",
    "productType": "DOWNLOAD",
    "pricing": { ... },
    // Additional fields omitted for brevity
  }
}
```

### To list all orders for the anonymous customer

Send a `GET` request to the `/orders` endpoint. 

```http
  GET https://api.service.com/v1/shoppers/me/orders
```

A successful response returns a 200 OK status code.

```json
{
  "orders": {
    "uri": "https://api.service.com/v1/shoppers/me/orders",
    "order": [
      {
        "id": 278666810197,
        "submissionDate": "2023-10-31T15:59:19.000Z",
        "orderState": "In Process"
      }
    ],
    "totalResults": "1",
    "totalResultPages": "1"
  }
}
```
