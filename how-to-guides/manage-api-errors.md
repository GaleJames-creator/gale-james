# Manage API errors

## Prerequisites

Before you start, make sure you:

- Have an integrated payment system

- Understand HTTP status codes

## Resolving API errors

Use these steps to identify and resolve API errors in your integration.

- Check the response headers for additional details, such as rate limits or warnings.
- Retry after waiting if a `5xx` error occurs. This helps your app recover from server issues.  
- Use the correct HTTP method—`GET`, `POST`, `PUT`, or `DELETE`—based on your action.
- Monitor [rate limits](./payment-api-rate-limits.md). Too many requests trigger a `429` error.  
- Review error codes in detail. Pay attention to any additional codes from the API.
- Test your API response with a REST client like Postman before adding it to your app.  
- Contact support if the same error recurs; some systems automatically report repeated errors.
