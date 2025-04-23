# POST /v1/tiktok/posts

Create and publish a new TikTok post using this endpoint.

## Endpoint

```
POST https://tikapis.vercel.app/v1/tiktok/posts
```

## Authentication

Requires API key authentication via the `X-API-Key` header.

## Request Parameters

### Headers

| Name | Required | Description |
| ---- | -------- | ----------- |
| `Content-Type` | Yes | Must be `application/json` |
| `X-API-Key` | Yes | Your API key |

### Body Parameters

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `videoUrl` | String | Yes | Direct URL to the video file to be uploaded |
| `caption` | String | Yes | Caption text for the TikTok post (max 2200 characters) |
| `hashtags` | Array | No | Array of hashtags to include (without # symbol) |
| `schedule` | ISO Date String | No | Future date/time to publish the post (ISO 8601 format) |
| `visibility` | String | No | Visibility setting (`public`, `friends`, `private`) - defaults to `public` |

## Example Request

```json
{
  "videoUrl": "https://example.com/videos/my-video.mp4",
  "caption": "Check out my awesome video!",
  "hashtags": ["tikapis", "automation", "coding"],
  "schedule": "2025-06-15T14:30:00Z",
  "visibility": "public"
}
```

## Response

### Success Response (200 OK)

```json
{
  "success": true,
  "data": {
    "id": "post_12345",
    "status": "scheduled",
    "scheduledAt": "2025-06-15T14:30:00Z",
    "videoUrl": "https://example.com/videos/my-video.mp4",
    "caption": "Check out my awesome video! #tikapis #automation #coding",
    "visibility": "public",
    "createdAt": "2025-06-10T08:15:22Z"
  }
}
```

### Error Response (400 Bad Request)

```json
{
  "success": false,
  "error": {
    "code": "invalid_request",
    "message": "The videoUrl provided is not accessible or is in an unsupported format."
  }
}
```

## Response Codes

| Status Code | Description |
| ----------- | ----------- |
| 200 | Success - Post created or scheduled |
| 400 | Bad Request - Invalid parameters |
| 401 | Unauthorized - Invalid API key |
| 403 | Forbidden - Insufficient permissions |
| 429 | Too Many Requests - Rate limit exceeded |
| 500 | Server Error - Something went wrong on our end |

## Supported Video Formats

- MP4 (recommended)
- MOV
- WebM
- AVI

## Video Requirements

- Maximum file size: 50MB
- Maximum duration: 3 minutes
- Recommended aspect ratios: 9:16 (portrait), 1:1 (square)
- Recommended resolution: At least 720p (1280×720)

## Notes

- Videos are processed asynchronously. The response indicates the post was successfully created or scheduled, but processing may take time.
- For immediate posting, omit the `schedule` parameter.
- Hashtags will be automatically formatted and appended to your caption.
- Rate limits apply to this endpoint. See [Rate Limits](rate-limits.md) for details.
