# Status codes  
Web services use status codes to tell you what happened. A`2xx` means success, `4xx` implies a problem with your request, and `5xx` means a server problem. You might also see an error code with more details. Knowing these codes helps you quickly resolve issues with your API requests.

## Best practices for API error codes
Follow these simple steps if you get an error code:
1. Check response headers for extra details such as rate limits or warnings.  
2. Retry after waiting if a `5xx` error occurs. This helps your app recover from server issues.  
3. Use the correct HTTP method—GET, POST, PUT, or DELETE—based on your action.
4. Monitor rate limits. Too many requests trigger a `429` error.  
5. Review error codes in detail. Pay attention to any additional codes from the API.
6. Test your API response with a REST client like Postman before adding it to your app.  
7. Contact support if the same error recurs, or create a support case; some systems report repeated errors automatically.
