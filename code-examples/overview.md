# Code Examples

This section provides practical examples of how to integrate with the TikAPIs platform in various programming languages.

## Available Examples

- [JavaScript/Node.js](javascript.md) - Examples using JavaScript and Node.js
- [Python](python.md) - Examples using Python
- [Other Languages](other-languages.md) - Examples in additional languages

## General Integration Flow

Regardless of the programming language you choose, the general flow for integrating with TikAPIs is:

1. **Authentication** - Include your API key in requests
2. **API Request** - Format and send your request to the appropriate endpoint
3. **Response Handling** - Process the API response
4. **Error Handling** - Implement proper error handling for various response codes

## Best Practices

### Secure API Key Storage

Always store your API keys securely:

```javascript
// In Node.js, use environment variables
const apiKey = process.env.TIKAPIS_API_KEY;
```

```python
# In Python, use environment variables
import os
api_key = os.environ.get("TIKAPIS_API_KEY")
```

### Implement Proper Error Handling

Always handle errors appropriately in your integration:

```javascript
fetch('https://tikapis.vercel.app/v1/tiktok/posts', {
  headers: { 'X-API-Key': apiKey }
})
.then(response => {
  if (!response.ok) {
    throw new Error(`API error: ${response.status}`);
  }
  return response.json();
})
.then(data => {
  // Handle successful response
})
.catch(error => {
  // Handle error properly
  console.error('Request failed:', error);
});
```

### Rate Limit Handling

Implement backoff logic to handle rate limiting:

```javascript
function fetchWithRetry(url, options, maxRetries = 3, delay = 1000) {
  return fetch(url, options)
    .then(response => {
      // If rate limited and retries remaining
      if (response.status === 429 && maxRetries > 0) {
        return new Promise(resolve => {
          setTimeout(() => {
            resolve(fetchWithRetry(url, options, maxRetries - 1, delay * 2));
          }, delay);
        });
      }
      return response;
    });
}
```

## Next Steps

Explore the language-specific examples to get started with your integration!

- [JavaScript Examples](javascript.md)
- [Python Examples](python.md)
