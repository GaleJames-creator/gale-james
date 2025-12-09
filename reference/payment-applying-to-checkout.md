# Applying a payment method to checkout  
This guide explains how to add a credit card and billing address as a payment method during checkout with JSON. If the request is successful, you will get a success status code. If there is an issue, the response will describe the problem. Common error responses include invalid card number, expired card, and invalid CVV. By understanding these potential errors, you can better anticipate and handle them in your integration.

## Endpoint

```json
	POST /checkout/{checkoutId}/payment-method
```

## Authentication

The `/payment-method` endpoint authorization requires a OAuth 2.0 `Bearer` token in the header as shown in the following example. 

```json
	Content-Type: application/json
	Authorization: Bearer {YOUR_TOKEN}
```

## Request body (JSON)

```json
{
	"paymentMethod": {
		"type": "credit_card",
		"cardNumber": "4111111111111111",
		"expirationMonth": "12",
		"expirationYear": "2027",
		"cvv": "123",
		"cardholderName": "Jane Doe"
	},
	"billingAddress": {
		"line1": "123 Main St",
		"city": "Anytown",
		"state": "CA",
		"postalCode": "12345",
		"country": "US"
	}
}
```
Here’s a summary table of the request parameters for the Apply Payment Method, including their types and whether they are required:

Request Parameter Summary

| Parameter	                   	| Location		| Type		| Required	| Description
| ----------------------------- | ------------- | --------- | --------- | --------------------------------------------- |  
| checkoutId	     			| path			| string	| Yes		| Unique identifier for the checkout			| 
| paymentMethod.type			| body (JSON)	| string	| Yes		| Type of payment method (e.g., credit_card)	| 
| paymentMethod.cardNumber		| body (JSON)	| string	| Yes		| Credit card number							| 
| paymentMethod.expirationMonth	| body (JSON)	| string	| Yes		| Card expiration month (MM)					| 
| paymentMethod.expirationYear	| body (JSON)	| string	| Yes		| Card expiration year (YYYY)					| 
| paymentMethod.cvv				| body (JSON)	| string	| Yes		| Card security code (CVV)						| 
| paymentMethod.cardholderName	| body (JSON)	| string	| Yes		| Name on the card								| 
| billingAddress.line1			| body (JSON)	| string	| Yes		| Street address								| 
| billingAddress.city			| body (JSON)	| string	| Yes		| City											| 
| billingAddress.state			| body (JSON)	| string	| Yes		| State or region								| 
| billingAddress.postalCode		| body (JSON)	| string	| Yes		| Postal code									| 
| billingAddress.country		| body (JSON)	| string	| Yes		| Two-letter country code (e.g., US)			| 

All parameters listed above are required for a successful request.

## Successful response (JSON)

```json
{
	"checkoutId": "abcd1234",
	"paymentStatus": "applied",
	"paymentMethod": {
		"type": "credit_card",
		"lastFour": "1111",
		"cardholderName": "Jane Doe"
	},
	"billingAddress": {
		"line1": "123 Main St",
		"city": "Anytown",
		"state": "CA",
		"postalCode": "12345",
		"country": "US"
	}
}
```

Error response (JSON)

```json
{
	"error": "Invalid card number."
}
```

If you get an error, see [Best practices for API error codes](.//payment-error-best-practices-for-handling-payment-api-errors.md) for instructions.
