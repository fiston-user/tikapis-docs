# Usage Tracking

TikAPIs provides comprehensive usage tracking to help you monitor your API consumption, analyze patterns, and optimize your integration. This page explains the available usage tracking features and how to interpret the data.

## Usage Dashboard

The usage dashboard offers a modern, visually-rich interface with interactive elements that help you understand your API usage patterns:

- **Interactive Cards**: Expandable cards showing usage summaries by category
- **Progress Bars**: Visual indicators of usage against your plan limits
- **Category Badges**: Color-coded badges for different API endpoints (auth, tiktok, etc.)
- **Data Visualization**: Charts and graphs showing usage trends over time

## Usage Metrics

The following metrics are tracked for your account:

### Request Metrics

| Metric | Description |
| ------ | ----------- |
| Total Requests | Total number of API requests made |
| Success Rate | Percentage of successful requests |
| Error Rate | Percentage of requests that resulted in errors |
| Average Response Time | Average time (ms) to process requests |

### Endpoint Metrics

| Metric | Description |
| ------ | ----------- |
| Endpoint Usage | Number of requests per endpoint |
| Endpoint Distribution | Percentage breakdown of endpoint usage |
| Top Endpoints | Most frequently used endpoints |
| Endpoint Errors | Error rates by endpoint |

### Time-based Metrics

| Metric | Description |
| ------ | ----------- |
| Hourly Usage | Requests per hour |
| Daily Usage | Requests per day |
| Weekly Trends | Usage patterns over weeks |
| Monthly Summary | Monthly usage totals and trends |

## Viewing Usage Data

### Real-time Dashboard

The real-time dashboard shows your current usage status with:

1. **Current Period Usage**: Visual indicators of your usage in the current billing period
2. **Rate Limit Status**: How close you are to hitting rate limits
3. **Active Endpoints**: Which endpoints you're actively using
4. **Error Tracking**: Any recent errors or issues

### Historical Data

View historical usage data with flexible time ranges:

1. **Date Range Selection**: Select custom time periods for analysis
2. **Comparison View**: Compare usage across different time periods
3. **Trend Analysis**: View long-term usage patterns
4. **Export Options**: Download usage data in CSV or JSON format

## Understanding Usage Reports

### Usage Breakdown by Category

Usage is categorized by the type of API operation:

- **Authentication**: Identity and token-related operations
- **TikTok Posts**: Content creation and retrieval
- **Analytics**: Data and metrics retrieval
- **System**: Configuration and status operations

Each category is color-coded for easy identification, with modern badges indicating the category type.

### Rate Limit Monitoring

Monitor your rate limit status with:

- **Visual Indicators**: Color-coded gauges showing limit consumption
- **Time to Reset**: Countdown to when rate limits will reset
- **Historical Violations**: Record of when rate limits were exceeded
- **Limit Adjustment Recommendations**: Suggestions for plan upgrades based on usage

## Usage Alerts

Set up alerts to be notified when:

1. **Approaching Limits**: When you reach a specified percentage of your plan limits
2. **Rate Limit Warnings**: When you're making requests too quickly
3. **Unusual Activity**: When usage patterns deviate significantly from normal
4. **Error Spikes**: When error rates exceed normal thresholds

Alerts can be delivered via:
- Email
- Dashboard notifications
- Webhooks (for integration with your monitoring systems)

## Optimizing Your Usage

Based on your usage data, consider these optimization strategies:

1. **Implement Caching**: Cache responses that don't change frequently
2. **Batch Operations**: Combine multiple operations when possible
3. **Schedule During Off-peaks**: Schedule bulk operations during periods of lower usage
4. **Monitor Error Rates**: Address endpoints with high error rates
5. **Upgrade Strategically**: Use usage data to determine when to upgrade your plan

## API for Usage Data

Access your usage data programmatically through our usage endpoints:

```
GET /v1/account/usage
```

Parameters include:

| Parameter | Description | Default |
| --------- | ----------- | ------- |
| `startDate` | Start date for usage data | 30 days ago |
| `endDate` | End date for usage data | Today |
| `interval` | Data interval (hourly, daily, weekly, monthly) | daily |
| `category` | Filter by category | All categories |

## Related Resources

- [Rate Limits](../api-reference/rate-limits.md) - Understanding usage limitations
- [Dashboard Overview](dashboard.md) - Complete dashboard guide
- [API Keys](../getting-started/api-keys.md) - Managing authentication
