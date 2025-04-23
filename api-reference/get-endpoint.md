# GET /v1/tiktok/posts

Retrieve information about your TikTok posts with this endpoint.

## Endpoint

```
GET https://tikapis.vercel.app/v1/tiktok/posts
```

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
| `status` | String | No | Filter by post status (`pending`, `scheduled`, `published`, `failed`) |
| `startDate` | ISO Date String | No | Filter posts created on or after this date |
| `endDate` | ISO Date String | No | Filter posts created on or before this date |
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
  "success": true,
  "data": [
    {
      "id": "post_12345",
      "status": "published",
      "videoUrl": "https://example.com/videos/my-video.mp4",
      "caption": "Check out my awesome video! #tikapis #automation #coding",
      "visibility": "public",
      "publishedAt": "2025-06-15T14:30:00Z",
      "createdAt": "2025-06-10T08:15:22Z",
      "metrics": {
        "views": 1240,
        "likes": 89,
        "comments": 12,
        "shares": 5
      }
    },
    {
      "id": "post_12346",
      "status": "scheduled",
      "videoUrl": "https://example.com/videos/another-video.mp4",
      "caption": "Coming soon! #tikapis",
      "visibility": "public",
      "scheduledAt": "2025-06-18T10:00:00Z",
      "createdAt": "2025-06-10T09:22:45Z"
    }
  ],
  "meta": {
    "pagination": {
      "total": 45,
      "pages": 5,
      "current": 1,
      "limit": 10
    }
  }
}
```

### Error Response (401 Unauthorized)

```json
{
  "success": false,
  "error": {
    "code": "unauthorized",
    "message": "Invalid API key provided."
  }
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
| `pending` | Post is being processed and prepared for publishing |
| `scheduled` | Post is scheduled to be published at a future time |
| `published` | Post has been successfully published to TikTok |
| `failed` | Post failed to publish due to an error |

## Metrics Availability

Metrics data (views, likes, comments, shares) is only available for posts with `published` status and may have a delay of up to 24 hours after publishing.

## Notes

- Results are sorted by creation date in descending order (newest first).
- For efficient API usage, use the pagination parameters.
- Rate limits apply to this endpoint. See [Rate Limits](rate-limits.md) for details.
