# Authentication

All requests to the TikAPIs platform must be authenticated using API keys. This guide explains how to authenticate your requests correctly.

{% hint style="info" %}
Your API keys carry many privileges, so be sure to keep them secure! Do not share your API keys in publicly accessible areas such as GitHub, client-side code, or in your public repositories.
{% endhint %}

## Using API Keys

To authenticate your API requests, you need to include your API key in the headers of your requests.

### API Key Header

Add the following header to all your API requests:

```
X-API-Key: your_api_key_here
```

### Example Request with Authentication

```bash
curl -X GET "https://tikapis.vercel.app/v1/tiktok/posts" \
     -H "X-API-Key: your_api_key_here"
```

## Security Best Practices

1. **Keep keys secure** - Never expose your API keys in client-side code or public repositories
2. **Environment variables** - Store your API keys in environment variables rather than hardcoding them
3. **Restrict access** - Only share API keys with team members who need them
4. **Rotate keys** - Periodically rotate your API keys for enhanced security
5. **Use separate keys** - Use different API keys for development and production environments

## Rate Limiting

API keys are subject to rate limits. If you exceed your rate limits, your requests will be rejected. See our [Rate Limits](../api-reference/rate-limits.md) page for more information.

## Next Steps

* [Generate API Keys](api-keys.md)
* [Explore API Endpoints](../api-reference/overview.md)
