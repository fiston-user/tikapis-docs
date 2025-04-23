# API Reference

This section provides detailed information about all available API endpoints in TikAPIs.

## Base URL

All API requests should use the following base URL:

```
https://tikapis.vercel.app/v1
```

## Available Endpoints

| Endpoint | Method | Description |
| -------- | ------ | ----------- |
| [`/tiktok/posts`](post-endpoint.md) | POST | Create and publish a new TikTok post |
| [`/tiktok/posts`](get-endpoint.md) | GET | Get information about your TikTok posts |

## Request Format

Requests should be formatted according to REST principles. The API accepts JSON for request bodies and returns JSON responses.

### Headers

All requests must include these headers:

| Header | Value | Description |
| ------ | ----- | ----------- |
| `Content-Type` | `application/json` | Specifies the format of the request body |
| `X-API-Key` | `your_api_key` | Your API authentication key |

## Response Format

All API responses are returned in JSON format. Each response includes:

- HTTP status code indicating success or failure
- Response body with requested data or error details

### Success Response Structure

```json
{
  "success": true,
  "data": { ... },
  "meta": { ... }
}
```

### Error Response Structure

```json
{
  "success": false,
  "error": {
    "code": "error_code",
    "message": "Human-readable error message"
  }
}
```

## Error Codes

| Status Code | Error Code | Description |
| ----------- | ---------- | ----------- |
| 400 | `invalid_request` | The request was malformed or missing required parameters |
| 401 | `unauthorized` | Missing or invalid API key |
| 403 | `forbidden` | The API key doesn't have permission for this action |
| 404 | `not_found` | The requested resource was not found |
| 429 | `rate_limit_exceeded` | You've exceeded your rate limit |
| 500 | `server_error` | An error occurred on the server |

## Pagination

For endpoints that return collections of items, pagination is supported using the following query parameters:

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `page` | Page number to request | 1 |
| `limit` | Number of items per page | 20 |

Paginated responses include a `meta` object with pagination details:

```json
{
  "success": true,
  "data": [ ... ],
  "meta": {
    "pagination": {
      "total": 45,
      "pages": 3,
      "current": 1,
      "limit": 20
    }
  }
}
```
