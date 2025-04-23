# API Keys

API keys are essential for authenticating your requests to TikAPIs. This guide explains how to manage your API keys.

## Generating an API Key

1. Log in to your TikAPIs account
2. Navigate to the **Dashboard > API Keys** section
3. Click on the **Generate New Key** button
4. Name your key to identify its purpose (e.g., "Development", "Production")
5. Copy your API key immediately – it will only be shown once!

{% hint style="warning" %}
API keys are only displayed once when created. If you lose your key, you'll need to generate a new one.
{% endhint %}

## Managing API Keys

Through the API Keys dashboard, you can:

* View active API keys
* Rename existing keys
* Check last usage timestamps
* Monitor usage metrics
* Revoke keys that are no longer needed

## Key Permissions

Currently, API keys provide full access to all available endpoints. In the future, we may introduce scoped permissions to allow more granular control.

## Usage Tracking

All API usage is tracked and can be viewed in your dashboard. This helps you monitor your API usage and stay within rate limits.

## Key Security

{% hint style="danger" %}
Never share your API keys or commit them to public repositories. Exposed keys should be revoked immediately.
{% endhint %}

Recommended practices:

* Store keys in environment variables
* Use secrets management tools for production environments
* Rotate keys periodically (every 90 days recommended)
* Set up monitoring for unusual API key usage

## Next Steps

* Learn about our [API endpoints](../api-reference/overview.md)
* Understand [rate limits](../api-reference/rate-limits.md)
* See [code examples](../code-examples/overview.md) for implementation
