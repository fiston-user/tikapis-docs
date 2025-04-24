# POST /v1/tiktok/posts

Create and publish a new TikTok post using this endpoint.

## Endpoint

```
POST https://tikapis.vercel.app/v1/tiktok/posts
```

## Prerequisites

Before using this endpoint, you must:
1. Connect your TikTok account in the dashboard
2. Ensure you have the required TikTok permissions (user.info.basic, video.upload)
3. Verify your TikTok account is active and in good standing

## Authentication

Requires API key authentication via the `X-API-Key` header.

## Request Parameters

### Headers

| Name | Required | Description |
| ---- | -------- | ----------- |
| `X-API-Key` | Yes | Your API key |

### Body Parameters (multipart/form-data)

| Parameter | Type | Required | Description |
| --------- | ---- | -------- | ----------- |
| `videoFile` | File | Yes | The video file to upload |
| `caption` | String | No | Caption text for the TikTok post (max 2200 characters) |

## Example Request

```bash
# Using curl with form-data
curl -X POST "https://tikapis.vercel.app/v1/tiktok/posts" \
     -H "X-API-Key: your_api_key_here" \
     -F "videoFile=@/path/to/your/video.mp4" \
     -F "caption=Check out my awesome video!"
```

### JavaScript/Fetch Example

```javascript
const form = new FormData();
form.append('videoFile', videoFileBlob, 'my-video.mp4');
form.append('caption', 'Check out my awesome video!');

fetch('https://tikapis.vercel.app/v1/tiktok/posts', {
  method: 'POST',
  headers: {
    'X-API-Key': 'your_api_key_here'
  },
  body: form
})
.then(response => response.json())
.then(data => console.log(data))
.catch(error => console.error('Error:', error));
```

## Response

### Success Response (202 Accepted)

```json
{
  "message": "Post accepted and is processing.",
  "publishId": "6956423584532654082",
  "postId": "cl9z2x3m80000qp9abcdef123"
}
```

**Note:** The API returns a 202 Accepted status rather than 200 OK because video processing occurs asynchronously.

## Post Status Lifecycle

When creating a TikTok post, it goes through the following statuses:

1. **Pending**: Initial state when the post is created in our database
2. **Processing**: Post has been submitted to TikTok and is being processed
3. **Published**: Post has been successfully published to TikTok
4. **Failed**: Post failed during processing or publishing

You can check the status of your post using the GET endpoint with the `postId` returned from the POST request.

### Error Response (400 Bad Request)

```json
{
  "error": "Missing or invalid videoFile in form data"
}
```

### Error Response (401 Unauthorized)

```json
{
  "error": "Failed to obtain valid TikTok token."
}
```

### Error Response (404 Not Found)

```json
{
  "error": "TikTok account not connected for this user."
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
