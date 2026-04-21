# Managing API errors

Use these steps to identify and resolve API errors in your integration.

## Prerequisites

Before you start, make sure you:

* Have an integrated payment system
* Understand HTTP status codes

## Steps

1. Check the response headers for additional details, such as rate limits or warnings.
2. Retry after waiting if a `5xx` error occurs. This helps your app recover from server issues.  
3. Use the correct HTTP method—`GET`, `POST`, `PUT`, or `DELETE`—based on your action.
4. Monitor [rate limits](./managing-api-rate-limits.md). Too many requests trigger a `429` error.  
5. Review error codes in detail. Pay attention to any additional codes from the API.
6. Test your API response with a REST client like Postman before adding it to your app.  
7. Contact support if the same error recurs; some systems automatically report repeated errors.

## Next steps

### How-to guides

* [Managing API rate limits](./managing-api-rate-limits.md)
* [Implementing retry logic for failed payments](./implementing-retry-logic.md)

---

Last updated: April 2026
