# Payment API rate limits

Managing API rate limits involves monitoring and controlling how often your application makes requests to an API to avoid exceeding the API provider's quota, maintain API stability, and ensure fair usage.

## Rate limit tiers

| Tier | Requests per minute | Use case |
|------|---------------------|----------|
| Standard | 100 | Most applications |
| Premium | 1,000 | High-volume merchants |

## Rate limit headers

Each API response includes your current rate limit details.

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1699564860
```

- `X-RateLimit-Limit`: Maximum requests allowed per minute.
- `X-RateLimit-Remaining`: Requests remaining in current window.
- `X-RateLimit-Reset`: Unix timestamp when the limit resets.

## Handling rate limits

If you go over the rate limit, the API returns a `429 Too Many Requests` response.

```json
{
  "error": {
    "type": "rate_limit_error",
    "code": "rate_limit_exceeded",
    "message": "You have exceeded your rate limit. Please retry after 60 seconds."
  }
}
```

---

Last updated: April 2026
