# JavaScript Examples

This page provides practical examples of how to integrate with TikAPIs using JavaScript and Node.js.

## Prerequisites

- Node.js installed on your system
- TikAPIs API key

## Setup

First, set up a basic Node.js project and install dependencies:

```bash
# Create a new project directory
mkdir tikapis-integration
cd tikapis-integration

# Initialize a new Node.js project
npm init -y

# Install dependencies
npm install node-fetch dotenv
```

Create a `.env` file to securely store your API key:

```
TIKAPIS_API_KEY=your_api_key_here
```

## Basic Setup

Create a file named `config.js` for your API configuration:

```javascript
// config.js
require('dotenv').config();

module.exports = {
  apiKey: process.env.TIKAPIS_API_KEY,
  baseUrl: 'https://tikapis.vercel.app/v1'
};
```

## Example: Fetch TikTok Posts

Create a file named `getPosts.js`:

```javascript
// getPosts.js
const fetch = require('node-fetch');
const config = require('./config');

/**
 * Fetch TikTok posts with optional filtering
 * @param {Object} options - Query parameters
 * @param {string} options.status - Filter by post status
 * @param {string} options.startDate - Start date in ISO format
 * @param {number} options.page - Page number
 * @param {number} options.limit - Results per page
 * @returns {Promise<Object>} - API response
 */
async function getPosts(options = {}) {
  try {
    // Build query string from options
    const queryParams = new URLSearchParams();
    for (const [key, value] of Object.entries(options)) {
      if (value) queryParams.append(key, value);
    }
    
    const queryString = queryParams.toString() ? `?${queryParams.toString()}` : '';
    const url = `${config.baseUrl}/tiktok/posts${queryString}`;
    
    // Make API request
    const response = await fetch(url, {
      method: 'GET',
      headers: {
        'X-API-Key': config.apiKey
      }
    });
    
    // Handle API response
    if (!response.ok) {
      // Get error details from response
      const errorData = await response.json();
      throw new Error(
        `API error: ${response.status} - ${errorData.error?.message || 'Unknown error'}`
      );
    }
    
    return await response.json();
  } catch (error) {
    console.error('Failed to fetch posts:', error.message);
    throw error;
  }
}

// Example usage
async function main() {
  try {
    // Get published posts, 10 per page
    const result = await getPosts({
      status: 'published',
      limit: 10
    });
    
    console.log(`Found ${result.data.length} posts:`);
    result.data.forEach(post => {
      console.log(`- ${post.id}: ${post.caption} (${post.status})`);
    });
    
    console.log(`\nPagination: Page ${result.meta.pagination.current} of ${result.meta.pagination.pages}`);
  } catch (error) {
    console.error('Error in main function:', error.message);
  }
}

main();
```

## Example: Create a TikTok Post

Create a file named `createPost.js`:

```javascript
// createPost.js
const fetch = require('node-fetch');
const config = require('./config');

/**
 * Create a new TikTok post
 * @param {Object} postData - Post data
 * @param {string} postData.videoUrl - URL to the video file
 * @param {string} postData.caption - Post caption
 * @param {Array<string>} postData.hashtags - Array of hashtags
 * @param {string} postData.schedule - ISO date string for scheduled posting
 * @param {string} postData.visibility - Post visibility setting
 * @returns {Promise<Object>} - API response
 */
async function createPost(postData) {
  try {
    // Validate required fields
    if (!postData.videoUrl) throw new Error('videoUrl is required');
    if (!postData.caption) throw new Error('caption is required');
    
    const url = `${config.baseUrl}/tiktok/posts`;
    
    // Make API request
    const response = await fetch(url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-API-Key': config.apiKey
      },
      body: JSON.stringify(postData)
    });
    
    // Handle API response
    if (!response.ok) {
      // Get error details from response
      const errorData = await response.json();
      throw new Error(
        `API error: ${response.status} - ${errorData.error?.message || 'Unknown error'}`
      );
    }
    
    return await response.json();
  } catch (error) {
    console.error('Failed to create post:', error.message);
    throw error;
  }
}

// Example usage
async function main() {
  try {
    // Create a new post
    const result = await createPost({
      videoUrl: 'https://example.com/videos/my-tiktok-video.mp4',
      caption: 'Check out this awesome video created with TikAPIs!',
      hashtags: ['tikapis', 'automation', 'developer'],
      visibility: 'public'
    });
    
    console.log('Post created successfully:');
    console.log(`ID: ${result.data.id}`);
    console.log(`Status: ${result.data.status}`);
    console.log(`Created at: ${result.data.createdAt}`);
  } catch (error) {
    console.error('Error in main function:', error.message);
  }
}

main();
```

## Example: Handling Rate Limits

Create a file named `rateLimitHandler.js` to implement backoff and retry logic:

```javascript
// rateLimitHandler.js
const fetch = require('node-fetch');
const config = require('./config');

/**
 * Fetch with exponential backoff for rate limit handling
 * @param {string} url - API endpoint URL
 * @param {Object} options - Fetch options
 * @param {number} maxRetries - Maximum number of retry attempts
 * @param {number} baseDelay - Base delay in milliseconds
 * @returns {Promise<Response>} - Fetch response
 */
async function fetchWithRetry(url, options, maxRetries = 3, baseDelay = 1000) {
  // Initialize retry counter
  let retries = 0;
  
  // Retry function
  async function attempt() {
    try {
      const response = await fetch(url, options);
      
      // If rate limited and we have retries remaining
      if (response.status === 429 && retries < maxRetries) {
        // Get reset time from headers if available
        const resetTime = response.headers.get('X-RateLimit-Reset');
        let waitTime;
        
        if (resetTime) {
          // Calculate wait time based on reset time
          const resetDate = new Date(parseInt(resetTime) * 1000);
          waitTime = Math.max(0, resetDate - new Date()) + 100; // Add 100ms buffer
        } else {
          // Use exponential backoff if reset time not available
          waitTime = baseDelay * Math.pow(2, retries);
        }
        
        console.log(`Rate limited. Retrying in ${waitTime}ms (Attempt ${retries + 1}/${maxRetries})`);
        
        // Increment retry counter
        retries++;
        
        // Wait and retry
        return new Promise(resolve => {
          setTimeout(() => resolve(attempt()), waitTime);
        });
      }
      
      return response;
    } catch (error) {
      // Handle network errors with retry
      if (retries < maxRetries) {
        const waitTime = baseDelay * Math.pow(2, retries);
        console.log(`Network error. Retrying in ${waitTime}ms (Attempt ${retries + 1}/${maxRetries})`);
        retries++;
        
        return new Promise(resolve => {
          setTimeout(() => resolve(attempt()), waitTime);
        });
      }
      
      throw error;
    }
  }
  
  return attempt();
}

// Example usage
async function getPosts() {
  try {
    const url = `${config.baseUrl}/tiktok/posts`;
    
    // Use the retry-enabled fetch
    const response = await fetchWithRetry(url, {
      method: 'GET',
      headers: {
        'X-API-Key': config.apiKey
      }
    });
    
    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(`API error: ${response.status} - ${errorData.error?.message || 'Unknown error'}`);
    }
    
    return await response.json();
  } catch (error) {
    console.error('Failed to fetch posts:', error.message);
    throw error;
  }
}

// Test the rate limit handling
async function main() {
  try {
    const result = await getPosts();
    console.log(`Successfully retrieved ${result.data.length} posts`);
  } catch (error) {
    console.error('Error in main function:', error.message);
  }
}

main();
```

## Using with Frontend Frameworks

### React Example

```jsx
import { useState, useEffect } from 'react';

function TikTokPostsList() {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    // Note: In a real application, you should NOT expose your API key in frontend code
    // This should be handled by your backend service
    async function fetchPosts() {
      try {
        const response = await fetch('https://tikapis.vercel.app/v1/tiktok/posts', {
          headers: {
            'X-API-Key': process.env.REACT_APP_TIKAPIS_KEY // Set in .env.local
          }
        });
        
        if (!response.ok) throw new Error('Failed to fetch posts');
        
        const data = await response.json();
        setPosts(data.data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }
    
    fetchPosts();
  }, []);
  
  if (loading) return <p>Loading posts...</p>;
  if (error) return <p>Error: {error}</p>;
  
  return (
    <div>
      <h2>Your TikTok Posts</h2>
      <ul>
        {posts.map(post => (
          <li key={post.id}>
            <h3>{post.caption}</h3>
            <p>Status: {post.status}</p>
            {post.status === 'published' && (
              <p>Metrics: {post.metrics.views} views, {post.metrics.likes} likes</p>
            )}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## Security Considerations

{% hint style="danger" %}
Never expose your API key in client-side JavaScript code. For frontend applications, always proxy your API requests through a backend service.
{% endhint %}

For React, Next.js or similar applications, create an API route on your server:
