
# Applying a payment method to checkout  
This example explains how to add a credit card and billing address as a payment method during checkout with JSON. If the request is successful, you will get a success status code. If there is an issue, the response will describe the problem. Common error responses include invalid card number, expired card, and invalid CVV. By understanding these potential errors, you can better anticipate and handle them in your integration.

##Endpoint:

```json
	POST /checkout/{checkoutId}/payment-method
```

##Headers:

```json
	Content-Type: application/json
	Authorization: Bearer YOUR_TOKEN
```

##Request Body (JSON):

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

##Successful Response (JSON):

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

Error Response (JSON):

```json
{
	"error": "Invalid card number."
}
```

If you get an error, see [Best practices for API error codes](/status-codes.md#best-practices-for-api-error-codes) for instructions.
