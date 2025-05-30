# 📚 API Documentation

Complete API reference for the AI-Powered Chart Generation System. This RESTful API provides endpoints for chart generation, export, analytics, and user management.

## 📋 Table of Contents

- [Getting Started](#getting-started)
- [Authentication](#authentication)
- [Chart Generation](#chart-generation)
- [Chart Management](#chart-management)
- [Export & Download](#export--download)
- [Analytics](#analytics)
- [User Management](#user-management)
- [Real-time Features](#real-time-features)
- [Error Handling](#error-handling)
- [Rate Limiting](#rate-limiting)
- [SDKs & Examples](#sdks--examples)

## 🚀 Getting Started

### Base URLs
- **Production**: `https://api.chartgen.ai`
- **Staging**: `https://staging-api.chartgen.ai`
- **Development**: `http://localhost:5000`

### API Version
Current API version: `v1`
All endpoints are prefixed with `/api/v1`

### Content Type
All requests should use `Content-Type: application/json`

### OpenAPI Specification
Interactive API documentation available at:
- **Swagger UI**: `{base_url}/docs`
- **ReDoc**: `{base_url}/redoc`
- **OpenAPI JSON**: `{base_url}/openapi.json`

## 🔐 Authentication

The API uses JWT (JSON Web Tokens) for authentication. All protected endpoints require a valid token in the Authorization header.

### Authentication Methods

#### 1. API Key Authentication
```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
     -H "Content-Type: application/json" \
     https://api.chartgen.ai/api/v1/charts
```

#### 2. OAuth2 Authentication
```bash
# Get access token
curl -X POST https://api.chartgen.ai/auth/oauth/token \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -d "grant_type=client_credentials&client_id=YOUR_CLIENT_ID&client_secret=YOUR_CLIENT_SECRET"

# Use access token
curl -H "Authorization: Bearer ACCESS_TOKEN" \
     https://api.chartgen.ai/api/v1/charts
```

### Token Management

#### Get API Key
```http
POST /auth/api-keys
Content-Type: application/json
Authorization: Bearer USER_TOKEN

{
  "name": "My Application",
  "permissions": ["charts:read", "charts:write", "exports:create"]
}
```

#### Response
```json
{
  "api_key": "ak_1234567890abcdef",
  "name": "My Application",
  "permissions": ["charts:read", "charts:write", "exports:create"],
  "created_at": "2024-05-30T18:52:16Z",
  "expires_at": null
}
```

## 📊 Chart Generation

### Analyze Prompt

Analyze a natural language prompt without generating the chart.

```http
POST /api/v1/charts/analyze
Content-Type: application/json
Authorization: Bearer YOUR_TOKEN

{
  "prompt": "Show me the top 5 geographic regions by revenue in a donut chart",
  "include_alternatives": true
}
```

#### Response
```json
{
  "original_prompt": "Show me the top 5 geographic regions by revenue in a donut chart",
  "cleaned_prompt": "top 5 geographic regions by revenue donut",
  "entities": {
    "numbers": ["5"],
    "chart_types": ["doughnut"],
    "metrics": ["revenue"],
    "ranking_terms": ["top"]
  },
  "intent": "ranking",
  "chart_type": "doughnut",
  "data_source": "geographic_data",
  "metrics": ["revenue"],
  "filters": {
    "limit": 5
  },
  "confidence_score": 0.92,
  "alternative_interpretations": [
    {
      "chart_type": "pie",
      "reason": "Alternative visualization for distribution",
      "confidence": 0.87
    }
  ],
  "complexity_level": "medium",
  "processing_time": 0.234
}
```

### Generate Chart

Generate a complete chart from natural language description.

```http
POST /api/v1/charts/generate
Content-Type: application/json
Authorization: Bearer YOUR_TOKEN

{
  "prompt": "Top 5 geographic regions by revenue in a donut chart",
  "user_id": "user_123",
  "organization_id": "org_456",
  "custom_styling": {
    "color_scheme": "rainbow",
    "theme": "dark"
  },
  "data_source_override": null
}
```

#### Response
```json
{
  "success": true,
  "chart_id": "chart_789abc",
  "config": {
    "chart_type": "doughnut",
    "title": "Top 5 Geographic Regions by Revenue",
    "data_source": "geographic_data",
    "metrics": ["revenue"],
    "filters": {
      "limit": 5
    },
    "color_scheme": "rainbow",
    "intent": "ranking",
    "confidence": 0.92
  },
  "query": "SELECT region, revenue FROM geographic_data ORDER BY revenue DESC LIMIT 5",
  "data": [
    {
      "region": "North America",
      "revenue": 125000
    },
    {
      "region": "Asia Pacific",
      "revenue": 156000
    },
    {
      "region": "Europe",
      "revenue": 98000
    },
    {
      "region": "Latin America",
      "revenue": 67000
    },
    {
      "region": "Middle East",
      "revenue": 45000
    }
  ],
  "execution_time": 0.023,
  "generation_time": 1.234,
  "message": "Chart generated successfully"
}
```

### Batch Generate Charts

Generate multiple charts from a list of prompts.

```http
POST /api/v1/charts/batch
Content-Type: application/json
Authorization: Bearer YOUR_TOKEN

[
  "Sales by region bar chart",
  "Monthly revenue line chart for 2024",
  "Customer age distribution pie chart"
]
```

#### Response
```json
[
  {
    "success": true,
    "chart_id": "chart_001",
    "config": { ... },
    "prompt_index": 0
  },
  {
    "success": true,
    "chart_id": "chart_002",
    "config": { ... },
    "prompt_index": 1
  },
  {
    "success": false,
    "error": "Insufficient data for analysis",
    "prompt_index": 2
  }
]
```

## 📋 Chart Management

### List Charts

Get paginated list of user's charts.

```http
GET /api/v1/charts?page=1&per_page=20&search=sales&chart_type=bar
Authorization: Bearer YOUR_TOKEN
```

#### Query Parameters
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number (1-based) |
| `per_page` | integer | 20 | Items per page (max 100) |
| `search` | string | null | Search in titles and prompts |
| `chart_type` | string | null | Filter by chart type |
| `organization_id` | string | null | Filter by organization |
| `is_public` | boolean | null | Filter public/private charts |
| `created_after` | string | null | ISO date filter |
| `created_before` | string | null | ISO date filter |

#### Response
```json
{
  "charts": [
    {
      "id": "chart_123",
      "title": "Sales Performance Q4",
      "chart_type": "bar",
      "created_at": "2024-05-30T18:52:16Z",
      "updated_at": "2024-05-30T19:15:22Z",
      "render_count": 15,
      "is_public": false,
      "created_by": "user_123",
      "organization_id": "org_456"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total_items": 45,
    "total_pages": 3,
    "has_next": true,
    "has_previous": false
  }
}
```

### Get Chart Details

Get detailed information about a specific chart.

```http
GET /api/v1/charts/{chart_id}?include_data=false
Authorization: Bearer YOUR_TOKEN
```

#### Query Parameters
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `include_data` | boolean | false | Include raw chart data |
| `include_query` | boolean | false | Include SQL query |
| `include_analytics` | boolean | false | Include usage analytics |

#### Response
```json
{
  "id": "chart_123",
  "title": "Sales Performance Q4",
  "description": "Quarterly sales analysis by region",
  "config": {
    "chart_type": "bar",
    "color_scheme": "cool",
    "styling": { ... }
  },
  "metadata": {
    "created_at": "2024-05-30T18:52:16Z",
    "created_by": "user_123",
    "organization_id": "org_456",
    "prompt": "Show me Q4 sales by region",
    "confidence_score": 0.89,
    "processing_time": 0.456
  },
  "usage": {
    "render_count": 15,
    "export_count": 3,
    "last_accessed": "2024-05-30T20:15:00Z"
  },
  "permissions": {
    "can_edit": true,
    "can_delete": true,
    "can_share": true,
    "can_export": true
  }
}
```

### Update Chart

Update chart configuration or metadata.

```http
PUT /api/v1/charts/{chart_id}
Content-Type: application/json
Authorization: Bearer YOUR_TOKEN

{
  "title": "Updated Chart Title",
  "description": "New description",
  "config": {
    "color_scheme": "warm",
    "custom_styling": {
      "background_color": "#f5f5f5"
    }
  },
  "is_public": false,
  "tags": ["sales", "q4", "performance"]
}
```

### Delete Chart

Delete a chart permanently.

```http
DELETE /api/v1/charts/{chart_id}
Authorization: Bearer YOUR_TOKEN
```

#### Response
```json
{
  "success": true,
  "message": "Chart deleted successfully",
  "chart_id": "chart_123"
}
```

## 📤 Export & Download

### Export Chart

Export a chart in various formats.

```http
POST /api/v1/charts/{chart_id}/export
Content-Type: application/json
Authorization: Bearer YOUR_TOKEN

{
  "format": "png",
  "quality": "high",
  "include_data": true,
  "include_query": false,
  "custom_styling": {
    "width": 1920,
    "height": 1080,
    "background_color": "#ffffff"
  }
}
```

#### Supported Formats
| Format | Description | Quality Levels |
|--------|-------------|----------------|
| `png` | Raster image | low (72 DPI), medium (150 DPI), high (300 DPI) |
| `svg` | Vector image | N/A |
| `pdf` | Document | low, medium, high |
| `html` | Interactive web | N/A |
| `excel` | Spreadsheet | N/A |
| `pptx` | PowerPoint | N/A |

#### Response
```json
{
  "success": true,
  "format": "png",
  "data": "iVBORw0KGgoAAAANSUhEUgAA...", // Base64 encoded
  "filename": "chart_123.png",
  "size_bytes": 156789,
  "download_url": "https://api.chartgen.ai/downloads/temp_abc123.png",
  "expires_at": "2024-05-30T19:52:16Z"
}
```

### Download Export

Download a previously generated export.

```http
GET /api/v1/downloads/{download_token}
Authorization: Bearer YOUR_TOKEN
```

### Bulk Export

Export multiple charts at once.

```http
POST /api/v1/charts/bulk-export
Content-Type: application/json
Authorization: Bearer YOUR_TOKEN

{
  "chart_ids": ["chart_123", "chart_456", "chart_789"],
  "format": "pdf",
  "quality": "medium",
  "combine_into_single_file": true,
  "include_cover_page": true
}
```

## 📊 Analytics

### System Analytics

Get comprehensive system analytics and metrics.

```http
GET /api/v1/analytics?time_range=7d&include_trends=true
Authorization: Bearer YOUR_TOKEN
```

#### Query Parameters
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `time_range` | string | 7d | Time range (1d, 7d, 30d, 90d) |
| `include_trends` | boolean | false | Include trend analysis |
| `breakdown_by` | string | null | Breakdown by (user, organization, chart_type) |

#### Response
```json
{
  "metrics": {
    "total_charts_generated": 1247,
    "success_rate": 94.2,
    "average_generation_time": 0.847,
    "popular_chart_types": {
      "bar": 456,
      "line": 321,
      "pie": 234,
      "doughnut": 156
    },
    "popular_intents": {
      "ranking": 387,
      "trend": 298,
      "distribution": 267,
      "comparison": 195
    },
    "user_engagement": {
      "active_users": 89,
      "avg_charts_per_user": 14.0,
      "retention_rate": 67.4
    }
  },
  "trends": [
    {
      "date": "2024-05-24",
      "charts_created": 45,
      "success_rate": 95.6,
      "avg_generation_time": 0.789
    }
  ],
  "insights": [
    {
      "type": "info",
      "message": "Chart generation performance improved by 12% this week"
    }
  ],
  "generated_at": "2024-05-30T18:52:16Z"
}
```

### User Analytics

Get analytics for a specific user or organization.

```http
GET /api/v1/analytics/users/{user_id}?time_range=30d
Authorization: Bearer YOUR_TOKEN
```

### Chart Performance

Get performance metrics for specific charts.

```http
GET /api/v1/analytics/charts/{chart_id}/performance
Authorization: Bearer YOUR_TOKEN
```

## 👥 User Management

### Get Current User

Get information about the authenticated user.

```http
GET /api/v1/users/me
Authorization: Bearer YOUR_TOKEN
```

#### Response
```json
{
  "id": "user_123",
  "email": "user@example.com",
  "username": "user123",
  "first_name": "John",
  "last_name": "Doe",
  "role": "user",
  "organization_id": "org_456",
  "preferences": {
    "default_chart_type": "bar",
    "color_scheme": "cool",
    "theme": "light"
  },
  "subscription": {
    "tier": "pro",
    "charts_remaining": 850,
    "exports_remaining": 95
  },
  "created_at": "2024-01-15T10:30:00Z",
  "last_login": "2024-05-30T18:45:00Z"
}
```

### Update User Preferences

Update user preferences and settings.

```http
PUT /api/v1/users/me/preferences
Content-Type: application/json
Authorization: Bearer YOUR_TOKEN

{
  "default_chart_type": "line",
  "color_scheme": "warm",
  "theme": "dark",
  "email_notifications": true,
  "auto_save": true
}
```

## 🔄 Real-time Features

### WebSocket Connection

Connect to real-time collaboration features.

```javascript
const ws = new WebSocket('wss://api.chartgen.ai/ws');

// Authentication
ws.send(JSON.stringify({
  type: 'authenticate',
  token: 'YOUR_JWT_TOKEN'
}));

// Subscribe to chart updates
ws.send(JSON.stringify({
  type: 'subscribe_chart',
  chart_id: 'chart_123'
}));

// Listen for updates
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  
  if (data.type === 'chart_update') {
    // Handle chart update
    console.log('Chart updated:', data.chart_id);
  }
};
```

### WebSocket Events

| Event Type | Direction | Description |
|------------|-----------|-------------|
| `authenticate` | Client → Server | Authenticate WebSocket connection |
| `subscribe_chart` | Client → Server | Subscribe to chart updates |
| `chart_update` | Server → Client | Chart configuration changed |
| `user_joined` | Server → Client | User joined chart session |
| `user_left` | Server → Client | User left chart session |
| `cursor_move` | Bidirectional | User cursor position update |

## ❌ Error Handling

### Standard Error Response

All API errors follow this format:

```json
{
  "success": false,
  "error": "Human-readable error message",
  "error_code": "SPECIFIC_ERROR_CODE",
  "details": {
    "field": "Additional error context",
    "suggestion": "How to fix the error"
  },
  "timestamp": "2024-05-30T18:52:16Z",
  "request_id": "req_abc123"
}
```

### HTTP Status Codes

| Code | Meaning | Description |
|------|---------|-------------|
| 200 | OK | Request successful |
| 201 | Created | Resource created |
| 400 | Bad Request | Invalid request data |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource not found |
| 422 | Unprocessable Entity | Validation errors |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error |
| 503 | Service Unavailable | Temporary unavailability |

### Common Error Codes

| Error Code | Description | Solution |
|------------|-------------|----------|
| `INVALID_PROMPT` | Prompt is too short/long or invalid | Provide a clear, descriptive prompt |
| `RATE_LIMIT_EXCEEDED` | Too many requests | Wait or upgrade plan |
| `INSUFFICIENT_DATA` | Not enough data for chart | Check data source |
| `UNSUPPORTED_CHART_TYPE` | Requested chart type not supported | Use supported chart type |
| `EXPORT_FAILED` | Chart export failed | Retry or contact support |

## 🚦 Rate Limiting

### Rate Limits by Tier

| Tier | Requests/Hour | Burst Limit | Chart Generation/Day |
|------|---------------|-------------|---------------------|
| Free | 100 | 10 | 50 |
| Pro | 1,000 | 50 | 500 |
| Enterprise | 10,000 | 200 | Unlimited |

### Rate Limit Headers

```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1622547600
X-RateLimit-Retry-After: 3600
```

## 🛠️ SDKs & Examples

### Python SDK

```python
from chartgen import ChartGenClient

client = ChartGenClient(api_key="your_api_key")

# Generate chart
chart = client.generate_chart(
    prompt="Top 5 sales regions bar chart",
    user_id="user_123"
)

# Export chart
export = client.export_chart(
    chart_id=chart.id,
    format="png",
    quality="high"
)

# Save to file
with open("chart.png", "wb") as f:
    f.write(export.data)
```

### JavaScript SDK

```javascript
import { ChartGenClient } from '@chartgen/sdk';

const client = new ChartGenClient({
  apiKey: 'your_api_key',
  baseURL: 'https://api.chartgen.ai'
});

// Generate chart
const chart = await client.generateChart({
  prompt: 'Monthly revenue trend for 2024',
  userId: 'user_123'
});

// Export chart
const exportData = await client.exportChart(chart.id, {
  format: 'pdf',
  quality: 'high'
});
```

### cURL Examples

```bash
# Generate chart
curl -X POST "https://api.chartgen.ai/api/v1/charts/generate" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "Customer satisfaction scores by department",
    "user_id": "user_123"
  }'

# Export chart
curl -X POST "https://api.chartgen.ai/api/v1/charts/chart_123/export" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "format": "png",
    "quality": "high"
  }'
```

## 📞 Support

- **API Status**: [status.chartgen.ai](https://status.chartgen.ai)
- **Documentation**: [docs.chartgen.ai](https://docs.chartgen.ai)
- **Support Email**: api-support@chartgen.ai
- **GitHub Issues**: [Issues](https://github.com/myownipgit/AI-Powered-Chart-Generation-System/issues)

For additional examples and tutorials, visit our [API Examples Repository](https://github.com/chartgen/api-examples).
