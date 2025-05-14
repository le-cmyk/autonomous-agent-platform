# API Reference Documentation

## Overview

The platform provides a RESTful API that follows standard conventions:

- All requests and responses use JSON format
- Authentication uses JWT bearer tokens
- API versioning is included in the URL path
- HTTP status codes indicate success or failure
- Consistent error response format

## Base URL

```
https://api.agentplatform.example.com/v1
```

## Authentication

### Get Access Token

```
POST /auth/login
```

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "your-secure-password"
}
```

**Response**:
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 3600
}
```

For all other API endpoints, include the token in the Authorization header:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## Agent Management

### List Agents

```
GET /agents
```

Query parameters:
- `org_id` (optional): Filter by organization
- `status` (optional): Filter by status
- `type_id` (optional): Filter by agent type
- `limit` (optional): Maximum number of results
- `offset` (optional): Pagination offset

**Response**:
```json
{
  "data": [
    {
      "id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
      "name": "Daily Report Generator",
      "type": {
        "id": "1a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p",
        "name": "Report Generator"
      },
      "status": "active",
      "last_heartbeat": "2025-05-13T15:30:45Z",
      "created_at": "2025-05-01T12:00:00Z"
    },
    ...
  ],
  "pagination": {
    "total": 42,
    "limit": 10,
    "offset": 0
  }
}
```

### Get Agent Details

```
GET /agents/{agent_id}
```

**Response**:
```json
{
  "id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
  "name": "Daily Report Generator",
  "type_id": "1a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p",
  "org_id": "9876fedc-ba98-7654-3210-abcdef123456",
  "created_by": "0a1b2c3d-4e5f-6g7h-8i9j-0k1l2m3n4o5p",
  "api_key": "agt_1a2b3c4d5e6f7g8h9i0j",
  "webhook_url": "https://api.example.com/webhook/9b1deb4d",
  "config": {
    "report_type": "daily",
    "recipients": ["user@example.com"],
    "include_charts": true
  },
  "schedule": "0 9 * * 1-5",
  "status": "active",
  "last_heartbeat": "2025-05-13T15:30:45Z",
  "created_at": "2025-05-01T12:00:00Z",
  "updated_at": "2025-05-10T08:15:30Z"
}
```

### Create Agent

```
POST /agents
```

**Request Body**:
```json
{
  "name": "New Task Agent",
  "type_id": "1a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p",
  "config": {
    "task_type": "data-sync",
    "source": "database",
    "destination": "api"
  },
  "schedule": "*/30 * * * *",
  "use_webhook": true
}
```

**Response**:
```json
{
  "id": "8a7b6c5d-4e3f-2g1h-0i9j-8k7l6m5n4o3p",
  "name": "New Task Agent",
  "type_id": "1a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p",
  "org_id": "9876fedc-ba98-7654-3210-abcdef123456",
  "created_by": "0a1b2c3d-4e5f-6g7h-8i9j-0k1l2m3n4o5p",
  "api_key": "agt_8a7b6c5d4e3f2g1h0i9j",
  "webhook_url": "https://api.example.com/webhook/8a7b6c5d",
  "config": {
    "task_type": "data-sync",
    "source": "database",
    "destination": "api"
  },
  "schedule": "*/30 * * * *",
  "status": "inactive",
  "created_at": "2025-05-14T10:20:30Z",
  "updated_at": "2025-05-14T10:20:30Z"
}
```

### Update Agent

```
PATCH /agents/{agent_id}
```

**Request Body**:
```json
{
  "name": "Updated Task Agent",
  "config": {
    "task_type": "data-sync",
    "source": "database",
    "destination": "cloud-storage"
  },
  "schedule": "0 */2 * * *",
  "status": "active"
}
```

**Response**:
```json
{
  "id": "8a7b6c5d-4e3f-2g1h-0i9j-8k7l6m5n4o3p",
  "name": "Updated Task Agent",
  "config": {
    "task_type": "data-sync",
    "source": "database",
    "destination": "cloud-storage"
  },
  "schedule": "0 */2 * * *",
  "status": "active",
  "updated_at": "2025-05-14T11:25:30Z",
  "... other fields remain unchanged ..."
}
```

### Delete Agent

```
DELETE /agents/{agent_id}
```

**Response**:
```
HTTP/1.1 204 No Content
```

## Agent Monitoring

### Get Agent Logs

```
GET /agents/{agent_id}/logs
```

Query parameters:
- `status` (optional): Filter by status (success, error, warning)
- `start_date` (optional): Filter by date range start
- `end_date` (optional): Filter by date range end
- `limit` (optional): Maximum number of results
- `offset` (optional): Pagination offset

**Response**:
```json
{
  "data": [
    {
      "id": "1a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p",
      "agent_id": "8a7b6c5d-4e3f-2g1h-0i9j-8k7l6m5n4o3p",
      "status": "success",
      "message": "Task completed successfully",
      "details": {
        "items_processed": 250,
        "duration_ms": 1850
      },
      "started_at": "2025-05-14T12:00:00Z",
      "completed_at": "2025-05-14T12:00:01Z",
      "duration": 1850
    },
    ...
  ],
  "pagination": {
    "total": 156,
    "limit": 10,
    "offset": 0
  }
}
```

### Get Agent Metrics

```
GET /agents/{agent_id}/metrics
```

Query parameters:
- `metric` (required): Metric name (e.g., "execution_time", "memory_usage", "success_rate")
- `period` (optional): Time period (e.g., "1h", "24h", "7d", "30d")
- `interval` (optional): Aggregation interval (e.g., "1m", "5m", "1h")

**Response**:
```json
{
  "metric": "execution_time",
  "unit": "ms",
  "period": "24h",
  "interval": "1h",
  "data": [
    { "timestamp": "2025-05-14T00:00:00Z", "value": 1542 },
    { "timestamp": "2025-05-14T01:00:00Z", "value": 1638 },
    { "timestamp": "2025-05-14T02:00:00Z", "value": 1720 },
    ...
  ],
  "stats": {
    "min": 1320,
    "max": 2150,
    "avg": 1685,
    "p95": 1950
  }
}
```

## Alert Management

### Create Alert Configuration

```
POST /agents/{agent_id}/alerts
```

**Request Body**:
```json
{
  "name": "Execution Time Alert",
  "conditions": {
    "metric": "execution_time",
    "operator": "gt",
    "threshold": 5000,
    "duration": "5m"
  },
  "channels": {
    "email": ["ops@example.com"],
    "slack": "https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXX"
  },
  "enabled": true
}
```

**Response**:
```json
{
  "id": "2a3b4c5d-6e7f-8g9h-0i1j-2k3l4m5n6o7p",
  "agent_id": "8a7b6c5d-4e3f-2g1h-0i9j-8k7l6m5n4o3p",
  "name": "Execution Time Alert",
  "conditions": {
    "metric": "execution_time",
    "operator": "gt",
    "threshold": 5000,
    "duration": "5m"
  },
  "channels": {
    "email": ["ops@example.com"],
    "slack": "https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXX"
  },
  "enabled": true,
  "created_at": "2025-05-14T14:30:00Z",
  "updated_at": "2025-05-14T14:30:00Z"
}
```

## Public Agent API

### Agent Heartbeat

```
POST /webhook/{agent_id}/heartbeat
```

**Headers**:
```
X-API-Key: agt_1a2b3c4d5e6f7g8h9i0j
```

**Request Body**:
```json
{
  "status": "online",
  "metrics": {
    "memory_mb": 120,
    "cpu_percent": 5.2
  },
  "version": "1.2.0"
}
```

**Response**:
```json
{
  "acknowledged": true,
  "server_time": "2025-05-14T15:45:20Z"
}
```

### Report Execution Result

```
POST /webhook/{agent_id}/execution
```

**Headers**:
```
X-API-Key: agt_1a2b3c4d5e6f7g8h9i0j
```

**Request Body**:
```json
{
  "execution_id": "exec_3a4b5c6d7e8f",
  "status": "success",
  "message": "Task completed successfully",
  "details": {
    "items_processed": 250,
    "errors": 0,
    "warnings": 2
  },
  "started_at": "2025-05-14T16:00:00Z",
  "completed_at": "2025-05-14T16:00:45Z",
  "duration_ms": 45000
}
```

**Response**:
```json
{
  "acknowledged": true,
  "log_id": "1a2b3c4d-5e6f-7g8h-9i0j-1k2l3m4n5o6p",
  "server_time": "2025-05-14T16:01:00Z"
}
```

## Error Handling

All API errors follow a consistent format:

```json
{
  "error": {
    "code": "resource_not_found",
    "message": "The requested agent was not found",
    "details": {
      "resource_type": "agent",
      "resource_id": "invalid-uuid"
    }
  },
  "request_id": "req_1a2b3c4d5e6f"
}
```

Common error codes:
- `invalid_request`: Malformed request or invalid parameters
- `authentication_failed`: Invalid or expired credentials
- `unauthorized`: Authenticated but not allowed to access the resource
- `resource_not_found`: The requested resource does not exist
- `validation_error`: Request data failed validation
- `rate_limit_exceeded`: Too many requests in a given time period
- `server_error`: Unexpected server-side error
