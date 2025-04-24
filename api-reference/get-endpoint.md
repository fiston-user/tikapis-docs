# GET /v1/tiktok/posts

Retrieve information about your TikTok posts with this endpoint.

## Endpoint

```
GET https://tikapis.vercel.app/v1/tiktok/posts
```

## Prerequisites

Before using this endpoint, you must:
1. Have a connected TikTok account in the dashboard
2. Have created at least one TikTok post using the POST endpoint

## Authentication

Requires API key authentication via the `X-API-Key` header.

## Request Parameters

### Headers

| Name | Required | Description |
| ---- | -------- | ----------- |
| `X-API-Key` | Yes | Your API key |

### Query Parameters

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `status` | String | No | Filter by post status (`pending`, `processing`, `published`, `failed`) |
| `postId` | String | No | Get a specific post by its ID |
| `page` | Number | No | Page number for pagination (default: 1) |
| `limit` | Number | No | Number of results per page (default: 20, max: 100) |

## Example Request

```bash
curl -X GET "https://tikapis.vercel.app/v1/tiktok/posts?status=published&limit=10" \
     -H "X-API-Key: your_api_key_here"
```

## Response

### Success Response (200 OK)

```json
{
  "posts": [
    {
      "id": "cl9z2x3m80000qp9abcdef123",
      "userId": "user_abc123",
      "tiktokId": "6956423584532654082",
      "status": "published",
      "caption": "Check out my awesome video!",
      "createdAt": "2025-06-10T08:15:22Z",
      "publishedAt": "2025-06-10T08:25:12Z"
    },
    {
      "id": "cl9z2x3m90001qp9abcdef456",
      "userId": "user_abc123",
      "tiktokId": "6956423584532654083",
      "status": "processing",
      "caption": "Coming soon!",
      "createdAt": "2025-06-10T09:22:45Z"
    }
  ],
  "pagination": {
    "total": 12,
    "pages": 2,
    "current": 1,
    "limit": 10
  }
}
```

### Error Response (401 Unauthorized)

```json
{
  "error": "Invalid API key provided."
}
```

### Error Response (404 Not Found)

```json
{
  "error": "Post not found."
}
```

## Response Codes

| Status Code | Description |
| ----------- | ----------- |
| 200 | Success - Posts retrieved |
| 400 | Bad Request - Invalid parameters |
| 401 | Unauthorized - Invalid API key |
| 403 | Forbidden - Insufficient permissions |
| 429 | Too Many Requests - Rate limit exceeded |
| 500 | Server Error - Something went wrong on our end |

## Post Status Types

| Status | Description |
| ------ | ----------- |
| `pending` | Initial state when the post is created in our database |
| `processing` | Post has been submitted to TikTok and is being processed |
| `published` | Post has been successfully published to TikTok |
| `failed` | Post failed during processing or publishing |

## TikTok ID

The `tiktokId` field represents the unique identifier assigned by TikTok during the post creation process. This ID can be used to reference the post on the TikTok platform.

## Notes

- Results are sorted by creation date in descending order (newest first).
- For efficient API usage, use the pagination parameters.
- Rate limits apply to this endpoint. See [Rate Limits](rate-limits.md) for details.
