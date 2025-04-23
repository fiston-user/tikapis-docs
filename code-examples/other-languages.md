# Other Languages

This page provides examples of integrating with TikAPIs in languages other than JavaScript and Python.

## PHP Example

```php
<?php
// TikAPIs PHP Example

// Configuration
$apiKey = getenv('TIKAPIS_API_KEY'); // Use environment variable for security
$baseUrl = 'https://tikapis.vercel.app/v1';

/**
 * Get TikTok posts with optional filtering
 * 
 * @param string|null $status Filter by post status
 * @param string|null $startDate Start date in ISO format
 * @param int $page Page number
 * @param int $limit Results per page
 * @return array|null API response data
 */
function getPosts($status = null, $startDate = null, $page = 1, $limit = 20) {
    global $apiKey, $baseUrl;
    
    // Build query parameters
    $params = [];
    if (!empty($status)) {
        $params['status'] = $status;
    }
    if (!empty($startDate)) {
        $params['startDate'] = $startDate;
    }
    $params['page'] = $page;
    $params['limit'] = $limit;
    
    // Initialize cURL session
    $curl = curl_init();
    
    // Build URL with query parameters
    $url = $baseUrl . '/tiktok/posts';
    if (!empty($params)) {
        $url .= '?' . http_build_query($params);
    }
    
    // Set cURL options
    curl_setopt_array($curl, [
        CURLOPT_URL => $url,
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_HTTPHEADER => [
            'X-API-Key: ' . $apiKey,
            'Accept: application/json'
        ]
    ]);
    
    // Execute request
    $response = curl_exec($curl);
    $statusCode = curl_getinfo($curl, CURLINFO_HTTP_CODE);
    
    // Check for errors
    if (curl_errno($curl)) {
        echo 'cURL error: ' . curl_error($curl) . "\n";
        curl_close($curl);
        return null;
    }
    
    // Close cURL session
    curl_close($curl);
    
    // Parse response
    $responseData = json_decode($response, true);
    
    // Check for API errors
    if ($statusCode !== 200) {
        $errorMessage = isset($responseData['error']['message']) 
            ? $responseData['error']['message'] 
            : 'Unknown API error';
        echo "API error ({$statusCode}): {$errorMessage}\n";
        return null;
    }
    
    return $responseData;
}

/**
 * Create a new TikTok post
 * 
 * @param string $videoUrl URL to the video file
 * @param string $caption Post caption
 * @param array|null $hashtags Array of hashtags
 * @param string|null $schedule ISO date string for scheduled posting
 * @param string $visibility Post visibility setting
 * @return array|null API response data
 */
function createPost($videoUrl, $caption, $hashtags = null, $schedule = null, $visibility = 'public') {
    global $apiKey, $baseUrl;
    
    // Validate required fields
    if (empty($videoUrl) || empty($caption)) {
        echo "Error: videoUrl and caption are required\n";
        return null;
    }
    
    // Prepare post data
    $postData = [
        'videoUrl' => $videoUrl,
        'caption' => $caption,
        'visibility' => $visibility
    ];
    
    // Add optional fields
    if (!empty($hashtags)) {
        $postData['hashtags'] = $hashtags;
    }
    if (!empty($schedule)) {
        $postData['schedule'] = $schedule;
    }
    
    // Initialize cURL session
    $curl = curl_init();
    
    // Set cURL options
    curl_setopt_array($curl, [
        CURLOPT_URL => $baseUrl . '/tiktok/posts',
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_POST => true,
        CURLOPT_POSTFIELDS => json_encode($postData),
        CURLOPT_HTTPHEADER => [
            'Content-Type: application/json',
            'X-API-Key: ' . $apiKey,
            'Accept: application/json'
        ]
    ]);
    
    // Execute request
    $response = curl_exec($curl);
    $statusCode = curl_getinfo($curl, CURLINFO_HTTP_CODE);
    
    // Check for errors
    if (curl_errno($curl)) {
        echo 'cURL error: ' . curl_error($curl) . "\n";
        curl_close($curl);
        return null;
    }
    
    // Close cURL session
    curl_close($curl);
    
    // Parse response
    $responseData = json_decode($response, true);
    
    // Check for API errors
    if ($statusCode !== 200) {
        $errorMessage = isset($responseData['error']['message']) 
            ? $responseData['error']['message'] 
            : 'Unknown API error';
        echo "API error ({$statusCode}): {$errorMessage}\n";
        return null;
    }
    
    return $responseData;
}

// Example usage
function main() {
    // Example 1: Get posts
    echo "Fetching TikTok posts...\n";
    $postsResult = getPosts(status: 'published', limit: 5);
    
    if ($postsResult && $postsResult['success']) {
        $posts = $postsResult['data'];
        echo "Found " . count($posts) . " posts:\n";
        
        foreach ($posts as $post) {
            echo "- {$post['id']}: {$post['status']}\n";
        }
        
        // Display pagination info
        $pagination = $postsResult['meta']['pagination'] ?? [];
        if (!empty($pagination)) {
            echo "\nPage {$pagination['current']} of {$pagination['pages']}\n";
        }
    } else {
        echo "Failed to retrieve posts\n";
    }
    
    echo "\n";
    
    // Example 2: Create a post
    echo "Creating a new TikTok post...\n";
    $createResult = createPost(
        videoUrl: 'https://example.com/videos/my-tiktok-video.mp4',
        caption: 'Check out this awesome video created with TikAPIs!',
        hashtags: ['tikapis', 'automation', 'php'],
        visibility: 'public'
    );
    
    if ($createResult && $createResult['success']) {
        $postData = $createResult['data'];
        echo "Post created successfully:\n";
        echo "ID: {$postData['id']}\n";
        echo "Status: {$postData['status']}\n";
        echo "Created at: {$postData['createdAt']}\n";
    } else {
        echo "Failed to create post\n";
    }
}

// Run the example
main();
```

## Ruby Example

```ruby
# TikAPIs Ruby Example
require 'net/http'
require 'uri'
require 'json'

# Configuration
API_KEY = ENV['TIKAPIS_API_KEY'] # Use environment variable for security
BASE_URL = 'https://tikapis.vercel.app/v1'

# Class for TikAPIs integration
class TikApi
  def initialize(api_key, base_url)
    @api_key = api_key
    @base_url = base_url
  end
  
  # Get TikTok posts with optional filtering
  def get_posts(status: nil, start_date: nil, page: 1, limit: 20)
    # Build query parameters
    params = {}
    params[:status] = status if status
    params[:startDate] = start_date if start_date
    params[:page] = page
    params[:limit] = limit
    
    # Make API request
    request_get('/tiktok/posts', params)
  end
  
  # Create a new TikTok post
  def create_post(video_url:, caption:, hashtags: nil, schedule: nil, visibility: 'public')
    # Validate required fields
    raise ArgumentError, 'video_url is required' if video_url.nil? || video_url.empty?
    raise ArgumentError, 'caption is required' if caption.nil? || caption.empty?
    
    # Prepare request data
    post_data = {
      videoUrl: video_url,
      caption: caption,
      visibility: visibility
    }
    
    # Add optional fields
    post_data[:hashtags] = hashtags if hashtags
    post_data[:schedule] = schedule if schedule
    
    # Make API request
    request_post('/tiktok/posts', post_data)
  end
  
  private
  
  # Helper method for GET requests
  def request_get(endpoint, params = {})
    # Build URI with query parameters
    uri = URI.parse("#{@base_url}#{endpoint}")
    uri.query = URI.encode_www_form(params) unless params.empty?
    
    # Create HTTP request
    request = Net::HTTP::Get.new(uri)
    request['X-API-Key'] = @api_key
    request['Accept'] = 'application/json'
    
    # Execute request
    response = make_request(uri, request)
    
    # Parse and return response
    parse_response(response)
  end
  
  # Helper method for POST requests
  def request_post(endpoint, data)
    # Build URI
    uri = URI.parse("#{@base_url}#{endpoint}")
    
    # Create HTTP request
    request = Net::HTTP::Post.new(uri)
    request['Content-Type'] = 'application/json'
    request['X-API-Key'] = @api_key
    request['Accept'] = 'application/json'
    request.body = data.to_json
    
    # Execute request
    response = make_request(uri, request)
    
    # Parse and return response
    parse_response(response)
  end
  
  # Helper method to make HTTP requests
  def make_request(uri, request)
    # Create HTTP connection
    http = Net::HTTP.new(uri.host, uri.port)
    http.use_ssl = (uri.scheme == 'https')
    
    # Execute request
    begin
      http.request(request)
    rescue => e
      puts "Request error: #{e.message}"
      nil
    end
  end
  
  # Helper method to parse HTTP responses
  def parse_response(response)
    return nil if response.nil?
    
    begin
      # Parse JSON response
      result = JSON.parse(response.body)
      
      # Check for API errors
      if response.code.to_i != 200
        error_message = result['error']&.[]('message') || 'Unknown API error'
        puts "API error (#{response.code}): #{error_message}"
        nil
      else
        # Return successful response
        result
      end
    rescue JSON::ParserError => e
      puts "Error parsing response: #{e.message}"
      nil
    end
  end
 end

# Example usage
api = TikApi.new(API_KEY, BASE_URL)

# Example 1: Get posts
puts "Fetching TikTok posts..."
posts_result = api.get_posts(limit: 5)

if posts_result && posts_result['success']
  posts = posts_result['data']
  puts "Found #{posts.length} posts:"
  
  posts.each do |post|
    puts "- #{post['id']}: #{post['status']}"
  end
  
  # Display pagination info
  if posts_result['meta'] && posts_result['meta']['pagination']
    pagination = posts_result['meta']['pagination']
    puts "\nPage #{pagination['current']} of #{pagination['pages']}"
  end
else
  puts "Failed to retrieve posts"
end

puts ""

# Example 2: Create a post
puts "Creating a new TikTok post..."
begin
  create_result = api.create_post(
    video_url: 'https://example.com/videos/my-tiktok-video.mp4',
    caption: 'Check out this awesome video created with TikAPIs!',
    hashtags: ['tikapis', 'automation', 'ruby']
  )
  
  if create_result && create_result['success']
    post_data = create_result['data']
    puts "Post created successfully:"
    puts "ID: #{post_data['id']}"
    puts "Status: #{post_data['status']}"
    puts "Created at: #{post_data['createdAt']}"
  else
    puts "Failed to create post"
  end
rescue ArgumentError => e
  puts "Error: #{e.message}"
end
```

## Go Example

```go
// TikAPIs Go Example
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"
	"net/url"
	"os"
	"time"
)

// Configuration
const BaseURL = "https://tikapis.vercel.app/v1"

// ApiClient - Client for TikAPIs
type ApiClient struct {
	ApiKey  string
	BaseURL string
}

// PostsResponse - Structure for posts API response
type PostsResponse struct {
	Success bool             `json:"success"`
	Data    []Post           `json:"data"`
	Meta    PostsResponseMeta `json:"meta"`
}

// Post - Structure for a TikTok post
type Post struct {
	ID          string       `json:"id"`
	Status      string       `json:"status"`
	VideoURL    string       `json:"videoUrl"`
	Caption     string       `json:"caption"`
	Visibility  string       `json:"visibility"`
	PublishedAt string       `json:"publishedAt,omitempty"`
	ScheduledAt string       `json:"scheduledAt,omitempty"`
	CreatedAt   string       `json:"createdAt"`
	Metrics     *PostMetrics `json:"metrics,omitempty"`
}

// PostMetrics - Structure for post metrics
type PostMetrics struct {
	Views    int `json:"views"`
	Likes    int `json:"likes"`
	Comments int `json:"comments"`
	Shares   int `json:"shares"`
}

// PostsResponseMeta - Structure for response metadata
type PostsResponseMeta struct {
	Pagination PaginationMeta `json:"pagination"`
}

// PaginationMeta - Structure for pagination metadata
type PaginationMeta struct {
	Total   int `json:"total"`
	Pages   int `json:"pages"`
	Current int `json:"current"`
	Limit   int `json:"limit"`
}

// CreatePostRequest - Structure for creating a post
type CreatePostRequest struct {
	VideoURL   string   `json:"videoUrl"`
	Caption    string   `json:"caption"`
	Hashtags   []string `json:"hashtags,omitempty"`
	Schedule   string   `json:"schedule,omitempty"`
	Visibility string   `json:"visibility,omitempty"`
}

// CreatePostResponse - Structure for create post response
type CreatePostResponse struct {
	Success bool `json:"success"`
	Data    Post `json:"data"`
}

// ErrorResponse - Structure for error responses
type ErrorResponse struct {
	Success bool `json:"success"`
	Error   struct {
		Code    string `json:"code"`
		Message string `json:"message"`
	} `json:"error"`
}

// NewApiClient - Create a new API client
func NewApiClient(apiKey string) *ApiClient {
	return &ApiClient{
		ApiKey:  apiKey,
		BaseURL: BaseURL,
	}
}

// GetPosts - Get TikTok posts with optional filtering
func (c *ApiClient) GetPosts(status string, page, limit int) (*PostsResponse, error) {
	// Build query parameters
	params := url.Values{}
	if status != "" {
		params.Add("status", status)
	}
	params.Add("page", fmt.Sprintf("%d", page))
	params.Add("limit", fmt.Sprintf("%d", limit))
	
	// Build URL
	requestURL := fmt.Sprintf("%s/tiktok/posts?%s", c.BaseURL, params.Encode())
	
	// Create HTTP request
	req, err := http.NewRequest("GET", requestURL, nil)
	if err != nil {
		return nil, err
	}
	
	// Add headers
	req.Header.Add("X-API-Key", c.ApiKey)
	req.Header.Add("Accept", "application/json")
	
	// Execute request
	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()
	
	// Read response body
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		return nil, err
	}
	
	// Check for API errors
	if resp.StatusCode != 200 {
		var errorResp ErrorResponse
		if err := json.Unmarshal(body, &errorResp); err == nil {
			return nil, fmt.Errorf("API error (%d): %s", resp.StatusCode, errorResp.Error.Message)
		}
		return nil, fmt.Errorf("API error (%d): unknown error", resp.StatusCode)
	}
	
	// Parse response
	var postsResp PostsResponse
	if err := json.Unmarshal(body, &postsResp); err != nil {
		return nil, err
	}
	
	return &postsResp, nil
}

// CreatePost - Create a new TikTok post
func (c *ApiClient) CreatePost(videoURL, caption string, hashtags []string, schedule, visibility string) (*CreatePostResponse, error) {
	// Validate required fields
	if videoURL == "" {
		return nil, fmt.Errorf("videoURL is required")
	}
	if caption == "" {
		return nil, fmt.Errorf("caption is required")
	}
	
	// Create request data
	postData := CreatePostRequest{
		VideoURL:   videoURL,
		Caption:    caption,
		Hashtags:   hashtags,
		Schedule:   schedule,
		Visibility: visibility,
	}
	
	// Marshal to JSON
	jsonData, err := json.Marshal(postData)
	if err != nil {
		return nil, err
	}
	
	// Build URL
	requestURL := fmt.Sprintf("%s/tiktok/posts", c.BaseURL)
	
	// Create HTTP request
	req, err := http.NewRequest("POST", requestURL, bytes.NewBuffer(jsonData))
	if err != nil {
		return nil, err
	}
	
	// Add headers
	req.Header.Add("Content-Type", "application/json")
	req.Header.Add("X-API-Key", c.ApiKey)
	req.Header.Add("Accept", "application/json")
	
	// Execute request
	client := &http.Client{}
	resp, err := client.Do(req)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()
	
	// Read response body
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		return nil, err
	}
	
	// Check for API errors
	if resp.StatusCode != 200 {
		var errorResp ErrorResponse
		if err := json.Unmarshal(body, &errorResp); err == nil {
			return nil, fmt.Errorf("API error (%d): %s", resp.StatusCode, errorResp.Error.Message)
		}
		return nil, fmt.Errorf("API error (%d): unknown error", resp.StatusCode)
	}
	
	// Parse response
	var createResp CreatePostResponse
	if err := json.Unmarshal(body, &createResp); err != nil {
		return nil, err
	}
	
	return &createResp, nil
}

func main() {
	// Get API key from environment variable
	apiKey := os.Getenv("TIKAPIS_API_KEY")
	if apiKey == "" {
		fmt.Println("Error: TIKAPIS_API_KEY environment variable is not set")
		return
	}
	
	// Create API client
	client := NewApiClient(apiKey)
	
	// Example 1: Get posts
	fmt.Println("Fetching TikTok posts...")
	postsResp, err := client.GetPosts("", 1, 5)
	if err != nil {
		fmt.Printf("Error getting posts: %v\n", err)
	} else if postsResp.Success {
		fmt.Printf("Found %d posts:\n", len(postsResp.Data))
		
		for _, post := range postsResp.Data {
			fmt.Printf("- %s: %s\n", post.ID, post.Status)
		}
		
		// Display pagination info
		fmt.Printf("\nPage %d of %d\n",
			postsResp.Meta.Pagination.Current,
			postsResp.Meta.Pagination.Pages)
	} else {
		fmt.Println("Failed to retrieve posts")
	}
	
	fmt.Println()
	
	// Example 2: Create a post
	fmt.Println("Creating a new TikTok post...")
	createResp, err := client.CreatePost(
		"https://example.com/videos/my-tiktok-video.mp4",
		"Check out this awesome video created with TikAPIs!",
		[]string{"tikapis", "automation", "golang"},
		"",  // No schedule (post immediately)
		"public",
	)
	
	if err != nil {
		fmt.Printf("Error creating post: %v\n", err)
	} else if createResp.Success {
		fmt.Println("Post created successfully:")
		fmt.Printf("ID: %s\n", createResp.Data.ID)
		fmt.Printf("Status: %s\n", createResp.Data.Status)
		fmt.Printf("Created at: %s\n", createResp.Data.CreatedAt)
	} else {
		fmt.Println("Failed to create post")
	}
}
```

## REST APIs with cURL

If you prefer to use cURL directly or other REST API tools, here are examples of how to interact with the TikAPIs using cURL commands:

### Get TikTok Posts

```bash
# Get all posts (first page, default 20 per page)
curl -X GET "https://tikapis.vercel.app/v1/tiktok/posts" \
     -H "X-API-Key: your_api_key_here"

# Get published posts with pagination
curl -X GET "https://tikapis.vercel.app/v1/tiktok/posts?status=published&page=1&limit=10" \
     -H "X-API-Key: your_api_key_here"

# Get posts within a date range
curl -X GET "https://tikapis.vercel.app/v1/tiktok/posts?startDate=2025-01-01T00:00:00Z&endDate=2025-04-01T00:00:00Z" \
     -H "X-API-Key: your_api_key_here"
```

### Create a TikTok Post

```bash
# Create a post to be published immediately
curl -X POST "https://tikapis.vercel.app/v1/tiktok/posts" \
     -H "Content-Type: application/json" \
     -H "X-API-Key: your_api_key_here" \
     -d '{
         "videoUrl": "https://example.com/videos/my-video.mp4",
         "caption": "Check out this awesome video!",
         "hashtags": ["tikapis", "automation", "curl"],
         "visibility": "public"
     }'

# Schedule a post for future publishing
curl -X POST "https://tikapis.vercel.app/v1/tiktok/posts" \
     -H "Content-Type: application/json" \
     -H "X-API-Key: your_api_key_here" \
     -d '{
         "videoUrl": "https://example.com/videos/scheduled-video.mp4",
         "caption": "This will be posted later!",
         "hashtags": ["tikapis", "scheduled"],
         "schedule": "2025-06-15T14:30:00Z",
         "visibility": "public"
     }'
```

## Integration with Other Tools

### Postman

You can also use Postman to test and integrate with TikAPIs. Here's how to set it up:

1. **Create a Collection**:
   - Create a new collection named "TikAPIs"
   - Add a Collection Variable named `apiKey` with your API key
   - Add a Collection Variable named `baseUrl` with value `https://tikapis.vercel.app/v1`

2. **Set Up Environment**:
   - Create environments for different contexts (e.g., "Development", "Production")
   - Configure environment-specific variables if needed

3. **Create Requests**:
   - Create a "Get Posts" request with URL `{{baseUrl}}/tiktok/posts` and method `GET`
   - Create a "Create Post" request with URL `{{baseUrl}}/tiktok/posts` and method `POST`
   - Add header `X-API-Key: {{apiKey}}` to all requests
   - For POST requests, add header `Content-Type: application/json`

4. **Example Create Post Body**:
   ```json
   {
     "videoUrl": "https://example.com/videos/my-video.mp4",
     "caption": "Testing with Postman!",
     "hashtags": ["tikapis", "testing", "postman"],
     "visibility": "public"
   }
   ```

5. **Test Scripts**:
   Add a test script to validate responses:
   ```javascript
   pm.test("Status code is 200", function () {
       pm.response.to.have.status(200);
   });

   pm.test("Response is successful", function () {
       var jsonData = pm.response.json();
       pm.expect(jsonData.success).to.eql(true);
   });
   ```

## Next Steps

Regardless of which language or tool you choose for integration, remember to:

1. Always keep your API key secure
2. Implement proper error handling
3. Handle rate limits appropriately
4. Validate inputs before sending to the API
5. Consider implementing a proxy for frontend applications

For more examples or specific integrations, contact our support team.
