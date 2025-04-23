# Frequently Asked Questions

This page addresses common questions about using TikAPIs.

## General Questions

### What is TikAPIs?

TikAPIs is a platform that allows developers to automate TikTok content publishing through a simple REST API. Our service handles the complexities of TikTok integration so you can focus on creating great content.

### Is TikAPIs officially affiliated with TikTok?

No, TikAPIs is a third-party service that provides an API for TikTok integration. We are not officially affiliated with or endorsed by TikTok.

### What programming languages can I use with TikAPIs?

TikAPIs is a REST API that can be used with any programming language that can make HTTP requests. We provide code examples for JavaScript, Python, PHP, Ruby, and Go, but you can use any language of your choice.

## Account & Billing

### How do I create an account?

You can sign up for TikAPIs by visiting our website and clicking the "Sign Up" button. You'll need to provide a valid email address and create a password.

### What pricing plans are available?

We offer several tiers:

- **Free**: Limited requests and posts per day, perfect for testing and small projects
- **Pro**: Higher limits for individual creators and small businesses
- **Business**: Expanded capacity for agencies and larger businesses
- **Enterprise**: Custom solutions for high-volume needs

Visit our pricing page for current rates and limits.

### How do I upgrade my plan?

You can upgrade your plan from your account dashboard. Go to Settings > Billing to view available plans and complete the upgrade process.

## API Usage

### How do I get an API key?

After creating an account, navigate to the API Keys section in your dashboard and click "Generate New Key". Make sure to copy and securely store your key, as it will only be displayed once.

### Can I use TikAPIs in a mobile app?

Yes, but we strongly recommend proxying your API requests through a backend service to keep your API key secure. Never embed your API key directly in mobile applications.

### What happens if I exceed my rate limits?

If you exceed your rate limits, your requests will receive a 429 Too Many Requests response. You'll need to wait until your limits reset or upgrade to a higher plan for increased capacity.

## Content & Publishing

### What types of content can I publish?

Currently, TikAPIs supports publishing video content to TikTok. We support common video formats including MP4, MOV, WebM, and AVI.

### Can I schedule posts for the future?

Yes, you can schedule posts by including a `schedule` parameter with an ISO 8601 formatted date-time string in your POST request.

### How long does it take for a post to appear on TikTok?

After a successful API request, posts are typically processed and published within a few minutes. Scheduled posts will be published at the specified time.

### What are the video requirements?

- Maximum file size: 50MB
- Maximum duration: 3 minutes
- Recommended aspect ratios: 9:16 (portrait), 1:1 (square)
- Recommended resolution: At least 720p (1280×720)

## Troubleshooting

### Why am I getting authentication errors?

Authentication errors (401 Unauthorized) typically occur due to:
- Missing the `X-API-Key` header in your request
- Using an invalid or revoked API key
- Including the API key in the wrong format

Ensure you're including the header `X-API-Key: your_api_key_here` in all requests.

### Why did my post fail to publish?

Posts might fail to publish for several reasons:
- The video URL is inaccessible or points to an unsupported format
- The video exceeds size or duration limits
- The caption contains prohibited content or exceeds character limits
- There was a temporary issue with the TikTok API

Check the error message in the API response for specific details.

### How can I monitor the status of my posts?

You can use the `GET /v1/tiktok/posts` endpoint to retrieve information about your posts, including their current status. Filter by status to see only pending, scheduled, published, or failed posts.

## Support

### How do I get help if I'm having issues?

If you're experiencing problems with the API, you can:
1. Check this documentation for solutions to common issues
2. Visit our support center for additional resources
3. Contact our support team at support@tikapis.example.com

### Do you offer a Service Level Agreement (SLA)?

Yes, Business and Enterprise plans include an SLA with guaranteed uptime and support response times. Contact our sales team for details.

### Can I request a new feature?

We welcome feature requests! Please send your suggestions to feedback@tikapis.example.com or use the feedback form in your dashboard.
