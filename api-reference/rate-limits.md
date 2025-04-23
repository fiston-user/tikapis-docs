# Rate Limits

To ensure service stability and fair usage for all users, TikAPIs implements rate limiting on API requests.

## Rate Limit Structure

Rate limits are applied based on your account tier and are calculated per API key. Each endpoint has its own specific rate limits.

| Tier | Requests per minute | Requests per day | Posts per day |
| ---- | ------------------- | ---------------- | ------------- |
| Free | 30 | 1,000 | 5 |
| Pro | 60 | 5,000 | 20 |
| Business | 120 | 20,000 | 50 |
| Enterprise | Custom | Custom | Custom |

## Rate Limit Headers

Each API response includes headers that provide information about your current rate limit status:

| Header | Description |
| ------ | ----------- |
| `X-RateLimit-Limit` | The maximum number of requests allowed per period |
| `X-RateLimit-Remaining` | The number of requests remaining in the current period |
| `X-RateLimit-Reset` | The time at which the current rate limit window resets (in Unix time) |

## Example Headers

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1714589312
```

## Rate Limit Exceeded

When you exceed the rate limit, you'll receive a `429 Too Many Requests` response with a JSON error body:

```json
{
  "success": false,
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded. Please try again in 45 seconds."
  }
}
```

## Best Practices

To avoid hitting rate limits:

1. **Implement backoff logic** - When you receive a 429 response, wait before retrying
2. **Cache responses** - Store responses that don't change frequently
3. **Batch operations** - Group multiple operations into a single request when possible
4. **Monitor usage** - Use the [API Usage Dashboard](../resources/usage-tracking.md) to monitor your consumption
5. **Spread requests** - Distribute requests evenly rather than sending them all at once

## Usage Dashboard

Track your API usage in real-time through our [Dashboard](../resources/dashboard.md) which provides:

- Visual charts of daily and monthly usage
- Categorized consumption by endpoint
- Rate limit status and remaining quota
- Usage history and trends
- Alerts for approaching limits

## Upgrading Limits

If you consistently reach your rate limits, consider upgrading to a higher tier plan. Visit your account settings or contact our support team to discuss options.

## Enterprise Solutions

For high-volume needs, our Enterprise plan offers custom rate limits tailored to your specific requirements. [Contact us](mailto:enterprise@tikapis.example.com) to discuss Enterprise options.
