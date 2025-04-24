# Python Examples

This page provides practical examples of how to integrate with TikAPIs using Python.

## Prerequisites

- Python 3.6 or higher
- TikAPIs API key

## Setup

First, set up a Python virtual environment and install dependencies:

```bash
# Create a virtual environment
python -m venv venv

# Activate the virtual environment
# On Windows
# venv\Scripts\activate
# On macOS/Linux
source venv/bin/activate

# Install dependencies
pip install requests python-dotenv
```

Create a `.env` file to securely store your API key:

```
TIKAPIS_API_KEY=your_api_key_here
```

## Basic Setup

Create a file named `config.py` to handle configuration:

```python
# config.py
import os
from dotenv import load_dotenv

# Load environment variables from .env file
load_dotenv()

# Configuration
API_KEY = os.getenv("TIKAPIS_API_KEY")
BASE_URL = "https://tikapis.vercel.app/v1"

# Validate required configuration
if not API_KEY:
    raise ValueError("API_KEY is missing. Please set it in your .env file.")
```

## Example: Fetch TikTok Posts

Create a file named `get_posts.py`:

```python
# get_posts.py
import requests
import json
from config import API_KEY, BASE_URL


def get_posts(status=None, start_date=None, page=1, limit=20):
    """
    Fetch TikTok posts with optional filtering
    
    Args:
        status (str, optional): Filter by post status
        start_date (str, optional): Start date in ISO format
        page (int, optional): Page number
        limit (int, optional): Results per page
        
    Returns:
        dict: API response data
    """
    try:
        # Prepare request URL with query parameters
        url = f"{BASE_URL}/tiktok/posts"
        
        # Build query parameters
        params = {}
        if status:
            params["status"] = status
        if start_date:
            params["startDate"] = start_date
        params["page"] = page
        params["limit"] = limit
        
        # Set up headers
        headers = {
            "X-API-Key": API_KEY
        }
        
        # Make API request
        response = requests.get(url, headers=headers, params=params)
        
        # Raise exception for HTTP errors
        response.raise_for_status()
        
        # Parse and return JSON response
        return response.json()
    
    except requests.exceptions.HTTPError as http_err:
        # Handle HTTP errors
        try:
            error_data = response.json()
            error_message = error_data.get("error", {}).get("message", str(http_err))
            print(f"HTTP error: {response.status_code} - {error_message}")
        except ValueError:
            print(f"HTTP error: {http_err}")
        return None
    
    except requests.exceptions.RequestException as err:
        # Handle general request errors
        print(f"Request error: {err}")
        return None


def main():
    # Example: Get published posts
    result = get_posts(status="published", limit=10)
    
    if result and result.get("success"):
        posts = result.get("data", [])
        print(f"Found {len(posts)} posts:")
        
        for post in posts:
            print(f"- {post['id']}: {post['caption'][:50]}... ({post['status']})")
        
        # Display pagination info
        pagination = result.get("meta", {}).get("pagination", {})
        if pagination:
            print(f"\nPage {pagination.get('current')} of {pagination.get('pages')}")
    else:
        print("Failed to retrieve posts")


if __name__ == "__main__":
    main()
```

## Example: Create a TikTok Post

Create a file named `create_post.py`:

```python
# create_post.py
import requests
import json
from config import API_KEY, BASE_URL


def create_post(video_file_path, caption=None):
    """
    Create a new TikTok post
    
    Args:
        video_file_path (str): Path to the local video file to upload
        caption (str, optional): Caption text for the TikTok post
        
    Returns:
        dict: API response data
    """
    try:
        # Validate required fields
        if not video_file_path:
            raise ValueError("video_file_path is required")
        if not caption:
            raise ValueError("caption is required")
        
        # Validate video file exists
        if not os.path.exists(video_file_path):
            raise ValueError(f"Video file not found: {video_file_path}")
        
        # Prepare form data with the video file
        files = {
            'videoFile': (os.path.basename(video_file_path), open(video_file_path, 'rb'), 'video/mp4')
        }
        
        # Add caption to form data if provided
        data = {}
        if caption:
            data['caption'] = caption
        
        # Set up headers
        headers = {
            "X-API-Key": API_KEY
        }
        
        # Make API request with multipart form data
        response = requests.post(
            f"{BASE_URL}/tiktok/posts",
            headers=headers,
            files=files,
            data=data
        )
        
        # Raise exception for HTTP errors
        response.raise_for_status()
        
        # Parse and return JSON response
        return response.json()
    
    except ValueError as val_err:
        # Handle validation errors
        print(f"Validation error: {val_err}")
        return None
    
    except requests.exceptions.HTTPError as http_err:
        # Handle HTTP errors
        try:
            error_data = response.json()
            error_message = error_data.get("error", {}).get("message", str(http_err))
            print(f"HTTP error: {response.status_code} - {error_message}")
        except ValueError:
            print(f"HTTP error: {http_err}")
        return None
    
    except requests.exceptions.RequestException as err:
        # Handle general request errors
        print(f"Request error: {err}")
        return None


def main():
    # Example: Create a new post
    result = create_post(
        video_file_path="/path/to/your/video.mp4",
        caption="Check out this awesome video created with TikAPIs!"
    )
    
    if result and result.get("success"):
        print("Post created successfully:")
        print(f"Message: {result.get('message')}")
        print(f"Post ID: {result.get('postId')}")
        print(f"TikTok Publish ID: {result.get('publishId')}")
    else:
        print("Failed to create post")


if __name__ == "__main__":
    main()
```

## Example: Handling Rate Limits

Create a file named `rate_limit_handler.py` to implement backoff and retry logic:

```python
# rate_limit_handler.py
import requests
import time
import json
from datetime import datetime
from config import API_KEY, BASE_URL


class RateLimitHandler:
    """
    Handler for API requests with rate limit handling
    """
    def __init__(self, api_key, base_url, max_retries=3, base_delay=1.0):
        """
        Initialize the handler
        
        Args:
            api_key (str): TikAPIs API key
            base_url (str): Base URL for API requests
            max_retries (int): Maximum number of retry attempts
            base_delay (float): Base delay in seconds
        """
        self.api_key = api_key
        self.base_url = base_url
        self.max_retries = max_retries
        self.base_delay = base_delay
    
    def request(self, method, endpoint, **kwargs):
        """
        Make an API request with rate limit handling
        
        Args:
            method (str): HTTP method (get, post, etc.)
            endpoint (str): API endpoint path
            **kwargs: Additional arguments to pass to requests
            
        Returns:
            requests.Response: API response
        """
        # Ensure headers are set
        if 'headers' not in kwargs:
            kwargs['headers'] = {}
        kwargs['headers']['X-API-Key'] = self.api_key
        
        # Build full URL
        url = f"{self.base_url}{endpoint}"
        
        # Initialize retry counter
        retries = 0
        
        while True:
            try:
                # Make the request
                response = requests.request(method, url, **kwargs)
                
                # If not rate limited or out of retries, return response
                if response.status_code != 429 or retries >= self.max_retries:
                    return response
                
                # Handle rate limiting
                retries += 1
                
                # Try to get reset time from headers
                reset_time = response.headers.get('X-RateLimit-Reset')
                if reset_time:
                    try:
                        # Convert Unix timestamp to seconds to wait
                        reset_timestamp = int(reset_time)
                        current_timestamp = int(time.time())
                        wait_time = max(0, reset_timestamp - current_timestamp) + 0.1  # Add small buffer
                    except (ValueError, TypeError):
                        # Fallback to exponential backoff if header parsing fails
                        wait_time = self.base_delay * (2 ** (retries - 1))
                else:
                    # Use exponential backoff if reset header not available
                    wait_time = self.base_delay * (2 ** (retries - 1))
                
                print(f"Rate limited. Retrying in {wait_time:.2f} seconds (Attempt {retries}/{self.max_retries})")
                time.sleep(wait_time)
            
            except requests.exceptions.RequestException as err:
                # Handle network errors with retry
                if retries < self.max_retries:
                    retries += 1
                    wait_time = self.base_delay * (2 ** (retries - 1))
                    print(f"Request error: {err}. Retrying in {wait_time:.2f} seconds (Attempt {retries}/{self.max_retries})")
                    time.sleep(wait_time)
                else:
                    # Raise the exception if out of retries
                    raise
    
    def get_posts(self, status=None, page=1, limit=20):
        """
        Get TikTok posts with rate limit handling
        
        Args:
            status (str, optional): Filter by post status
            page (int): Page number
            limit (int): Results per page
            
        Returns:
            dict: Parsed JSON response
        """
        params = {}
        if status:
            params['status'] = status
        params['page'] = page
        params['limit'] = limit
        
        response = self.request('get', '/tiktok/posts', params=params)
        response.raise_for_status()
        return response.json()
    
    def create_post(self, video_url, caption, **kwargs):
        """
        Create a TikTok post with rate limit handling
        
        Args:
            video_url (str): URL to the video file
            caption (str): Post caption
            **kwargs: Additional post data
            
        Returns:
            dict: Parsed JSON response
        """
        post_data = {
            'videoUrl': video_url,
            'caption': caption,
            **kwargs
        }
        
        headers = {'Content-Type': 'application/json'}
        
        response = self.request(
            'post',
            '/tiktok/posts',
            headers=headers,
            data=json.dumps(post_data)
        )
        response.raise_for_status()
        return response.json()


def main():
    # Create handler
    api = RateLimitHandler(API_KEY, BASE_URL)
    
    try:
        # Example: Get posts with rate limit handling
        result = api.get_posts(limit=5)
        
        if result.get("success"):
            posts = result.get("data", [])
            print(f"Successfully retrieved {len(posts)} posts")
            
            for post in posts:
                print(f"- {post['id']}: {post['status']}")
        else:
            print("API request failed")
    
    except Exception as e:
        print(f"Error: {e}")


if __name__ == "__main__":
    main()
```

## Using with Web Frameworks

### Flask Example

Create a file named `flask_api_proxy.py`:

```python
from flask import Flask, request, jsonify
import requests
import os
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

# Configuration
API_KEY = os.getenv("TIKAPIS_API_KEY")
BASE_URL = "https://tikapis.vercel.app/v1"

app = Flask(__name__)

@app.route('/api/tiktok/posts', methods=['GET'])
def get_posts():
    """Proxy endpoint for getting TikTok posts"""
    try:
        # Forward query parameters
        params = {k: v for k, v in request.args.items()}
        
        # Make request to TikAPIs
        response = requests.get(
            f"{BASE_URL}/tiktok/posts",
            headers={"X-API-Key": API_KEY},
            params=params
        )
        
        # Return response to client
        return jsonify(response.json()), response.status_code
    
    except Exception as e:
        return jsonify({
            "success": False,
            "error": {"message": str(e)}
        }), 500

@app.route('/api/tiktok/posts', methods=['POST'])
def create_post():
    """Proxy endpoint for creating TikTok posts"""
    try:
        # Get JSON request data
        post_data = request.get_json()
        
        # Make request to TikAPIs
        response = requests.post(
            f"{BASE_URL}/tiktok/posts",
            headers={
                "Content-Type": "application/json",
                "X-API-Key": API_KEY
            },
            json=post_data
        )
        
        # Return response to client
        return jsonify(response.json()), response.status_code
    
    except Exception as e:
        return jsonify({
            "success": False,
            "error": {"message": str(e)}
        }), 500


if __name__ == '__main__':
    app.run(debug=True)
```

### FastAPI Example

Create a file named `fastapi_api_proxy.py`:

```python
from fastapi import FastAPI, HTTPException, Request, Depends
from fastapi.responses import JSONResponse
from pydantic import BaseModel, Field
from typing import List, Optional
import httpx
import os
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

# Configuration
API_KEY = os.getenv("TIKAPIS_API_KEY")
BASE_URL = "https://tikapis.vercel.app/v1"

# FastAPI app
app = FastAPI(title="TikAPIs Proxy")

# Pydantic models for validation
class CreatePostRequest(BaseModel):
    videoUrl: str
    caption: str
    hashtags: Optional[List[str]] = None
    schedule: Optional[str] = None
    visibility: Optional[str] = "public"


# HTTP client
async def get_client():
    async with httpx.AsyncClient() as client:
        yield client


@app.get("/api/tiktok/posts")
async def get_posts(request: Request, client: httpx.AsyncClient = Depends(get_client)):
    """Proxy endpoint for getting TikTok posts"""
    try:
        # Get query parameters
        params = dict(request.query_params)
        
        # Make request to TikAPIs
        response = await client.get(
            f"{BASE_URL}/tiktok/posts",
            headers={"X-API-Key": API_KEY},
            params=params
        )
        
        # Raise exception for HTTP errors
        response.raise_for_status()
        
        # Return response data
        return response.json()
    
    except httpx.HTTPStatusError as e:
        # Handle API errors
        try:
            error_data = e.response.json()
            raise HTTPException(
                status_code=e.response.status_code,
                detail=error_data
            )
        except ValueError:
            raise HTTPException(
                status_code=e.response.status_code,
                detail={"error": {"message": str(e)}}
            )
    
    except Exception as e:
        # Handle general errors
        raise HTTPException(
            status_code=500,
            detail={"error": {"message": str(e)}}
        )


@app.post("/api/tiktok/posts")
async def create_post(
    post: CreatePostRequest,
    client: httpx.AsyncClient = Depends(get_client)
):
    """Proxy endpoint for creating TikTok posts"""
    try:
        # Make request to TikAPIs
        response = await client.post(
            f"{BASE_URL}/tiktok/posts",
            headers={
                "Content-Type": "application/json",
                "X-API-Key": API_KEY
            },
            json=post.dict()
        )
        
        # Raise exception for HTTP errors
        response.raise_for_status()
        
        # Return response data
        return response.json()
    
    except httpx.HTTPStatusError as e:
        # Handle API errors
        try:
            error_data = e.response.json()
            raise HTTPException(
                status_code=e.response.status_code,
                detail=error_data
            )
        except ValueError:
            raise HTTPException(
                status_code=e.response.status_code,
                detail={"error": {"message": str(e)}}
            )
    
    except Exception as e:
        # Handle general errors
        raise HTTPException(
            status_code=500,
            detail={"error": {"message": str(e)}}
        )


if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

## Security Considerations

{% hint style="warning" %}
Never hardcode your API key directly in your code. Always use environment variables or a secure configuration system.
{% endhint %}

For production applications:

- Use a secrets management system
- Consider using a backend proxy to keep your API key secure
- Implement proper error handling and logging
- Use HTTPS for all connections
- Consider implementing a token rotation strategy
