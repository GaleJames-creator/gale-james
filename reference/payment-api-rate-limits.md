# Payment API rate limits

API rate limits control how many requests your app can send within a given time period. Watch your usage and spread out requests so you don’t go over the limit. This keeps the API working well and fair for all users.

## Rate limit tiers

| Tier     | Requests per minute | Use case              |
| -------- | ------------------- | --------------------- |
| Standard | 100                 | Most applications     |
| Premium  | 1,000               | High-volume merchants |

## Rate limit headers

Each API response includes your current rate limit details.

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1699564860
```

* `X-RateLimit-Limit`: Maximum requests allowed per minute.
* `X-RateLimit-Remaining`: Requests remaining in current window.
* `X-RateLimit-Reset`: Unix timestamp when the limit resets.

## Handling rate limits

If you go over the rate limit, the API returns a `429 Too Many Requests` response.

```json
{
  "error": {
    "type": "rate_limit_error",
    "code": "rate_limit_exceeded",
    "message": "You have exceeded your rate limit. Check the X-RateLimit-Reset header for when to retry."
  }
}
```

## Next steps

### How-to guides

* [Managing API rate limits](../how-to-guides/managing-api-rate-limits.md): How to implement retry logic and avoid rate limits.

---

Last updated: April 2026
