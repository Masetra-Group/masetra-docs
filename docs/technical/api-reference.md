---
layout: default
title: "API Reference"
description: "Comprehensive documentation for Masetra's RESTful API endpoints and integration capabilities"
---

## 📋 API Overview

Masetra provides a comprehensive RESTful API that allows integration with external systems, custom applications, and automated workflows. All API endpoints follow REST principles and return JSON responses.

### Base URL

https://api.masetra.com/v1/


### Authentication

All API requests require authentication using JWT (JSON Web Tokens).

```bash
# Obtain access token
POST /api/auth/token/
Content-Type: application/json

{
  "username": "your_username",
  "password": "your_password"
}

# Use token in subsequent requests
Authorization: Bearer YOUR_ACCESS_TOKEN
```

Response Format
All responses are returned in JSON format with standard HTTP status codes.

```json
{
  "status": "success",
  "data": {},
  "message": "Optional descriptive message"
}
```

🔐 Authentication Endpoints
POST /api/auth/token/
Obtain Access Token

Request:

```json
{
  "username": "string",
  "password": "string"
}
```

Response:

```json
{
  "access": "jwt_token_string",
  "refresh": "refresh_token_string",
  "user": {
    "id": 1,
    "username": "user@example.com",
    "first_name": "John",
    "last_name": "Doe"
  }
}
```

POST /api/auth/refresh/
Refresh Access Token

Request:

```json
{
  "refresh": "refresh_token_string"
}
```

Response:

```json
{
  "access": "new_jwt_token_string"
}
```

POST /api/auth/logout/
Logout User

Request:

```json
{
  "refresh": "refresh_token_string"
}
```

📋 Deviations API
GET /api/deviations/
List Deviations

Query Parameters:

page (integer): Page number
limit (integer): Items per page (default: 20)
status (string): Filter by status
department (string): Filter by department
date_from (date): Filter from date
date_to (date): Filter to date
Response:

```json
{
  "count": 100,
  "next": "https://api.masetra.com/v1/deviations/?page=2",
  "previous": null,
  "results": [
    {
      "id": 1,
      "deviation_number": "DEV-2024-0001",
      "title": "Equipment Calibration Deviation",
      "description": "Equipment found out of calibration during routine check",
      "category": "EQUIPMENT",
      "classification": "MAJOR",
      "department": "Quality Control",
      "status": "OPEN",
      "priority": "HIGH",
      "initiated_by": 5,
      "initiation_date": "2024-01-15T09:30:00Z",
      "due_date": "2024-01-22T09:30:00Z"
    }
  ]
}
```

POST /api/deviations/
Create New Deviation

Request:

```json
{
  "title": "string",
  "description": "string",
  "category": "PROCEDURAL|EQUIPMENT|MATERIAL|ENVIRONMENTAL|OTHER",
  "classification": "CRITICAL|MAJOR|MINOR",
  "department": "string",
  "location": "string",
  "discovered_date": "2024-01-15T09:30:00Z",
  "related_batch": "string",
  "related_product": "string",
  "priority": "LOW|MEDIUM|HIGH|URGENT"
}
```

GET /api/deviations/{id}/
Get Specific Deviation

Response:

```json
{
  "id": 1,
  "deviation_number": "DEV-2024-0001",
  "title": "Equipment Calibration Deviation",
  "description": "Equipment found out of calibration during routine check",
  "category": "EQUIPMENT",
  "classification": "MAJOR",
  "department": "Quality Control",
  "location": "Lab 3",
  "status": "OPEN",
  "priority": "HIGH",
  "initiated_by": {
    "id": 5,
    "username": "john.doe@example.com",
    "first_name": "John",
    "last_name": "Doe"
  },
  "initiation_date": "2024-01-15T09:30:00Z",
  "discovered_date": "2024-01-14T14:20:00Z",
  "due_date": "2024-01-22T09:30:00Z",
  "attachments": [
    {
      "id": 1,
      "file": "https://api.masetra.com/media/deviation_attachments/file.pdf",
      "uploaded_by": 5,
      "uploaded_at": "2024-01-15T10:15:00Z"
    }
  ],
  "investigation": {
    "id": 1,
    "root_cause_analysis": "string",
    "investigation_method": "string",
    "findings": "string",
    "completed_at": "2024-01-18T11:00:00Z"
  }
}
```

PUT /api/deviations/{id}/
Update Deviation

DELETE /api/deviations/{id}/
Delete Deviation

POST /api/deviations/{id}/attachments/
Upload Attachment

Request (multipart/form-data):
file: (binary file)
description: "Optional description"
🛠️ CAPA API
GET /api/capas/
List CAPAs

POST /api/capas/
Create New CAPA

Request:

```json
{
  "title": "string",
  "description": "string",
  "priority": "LOW|MEDIUM|HIGH|URGENT",
  "department": "string",
  "target_completion_date": "2024-01-30T09:30:00Z",
  "source_type": "DEVIATION|AUDIT|COMPLAINT|MANUAL",
  "source_id": 1
}
```

GET /api/capas/{id}/
Get Specific CAPA

PUT /api/capas/{id}/
Update CAPA

DELETE /api/capas/{id}/
Delete CAPA

👥 Users API
GET /api/users/
List Users

POST /api/users/
Create User

GET /api/users/{id}/
Get User Details

PUT /api/users/{id}/
Update User

DELETE /api/users/{id}/
Delete User


📊 Reporting API
GET /api/reports/deviations/
Deviations Report

Query Parameters:

period (string): daily|weekly|monthly|yearly
start_date (date): Report start date
end_date (date): Report end date
department (string): Filter by department
GET /api/reports/capas/
CAPAs Report

GET /api/reports/dashboard/
Dashboard Statistics

⚙️ Configuration API
GET /api/config/system/
System Configuration

PUT /api/config/system/
Update System Configuration

GET /api/config/roles/
List Roles

POST /api/config/roles/
Create Role

📈 Webhooks
Deviation Events
deviation.created
deviation.updated
deviation.closed
deviation.approved

CAPA Events
capa.created
capa.updated
capa.closed
capa.approved

Webhook Configuration

```json
{
  "url": "https://your-system.com/webhook",
  "events": ["deviation.created", "capa.closed"],
  "secret": "your_webhook_secret"
}
```

📞 Error Handling
HTTP Status Codes
200 OK: Successful GET, PUT, PATCH
201 Created: Successful POST
204 No Content: Successful DELETE
400 Bad Request: Invalid request data
401 Unauthorized: Authentication required
403 Forbidden: Insufficient permissions
404 Not Found: Resource not found
409 Conflict: Resource conflict
500 Internal Server Error: Server error

Error Response Format

```json
{
  "status": "error",
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request data is invalid",
    "details": [
      {
        "field": "title",
        "message": "This field is required."
      }
    ]
  }
}
```

🔧 Rate Limiting
API requests are rate-limited to prevent abuse:

Anonymous requests: 100 requests/hour
Authenticated requests: 1000 requests/hour
Enterprise accounts: Configurable limits
Rate limit headers:

X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200


📚 SDKs and Libraries
Official SDKs
Python: pip install masetra-sdk
JavaScript: npm install @masetra/sdk
Java: Maven dependency available
.NET: NuGet package available

Example Usage (Python)
```python
from masetra_sdk import MasetraClient

client = MasetraClient(
    base_url="https://api.masetra.com/v1/",
    api_key="your_api_key"
)
```

# Create deviation
deviation = client.deviations.create({
    "title": "Test Deviation",
    "description": "Sample deviation for testing",
    "category": "PROCEDURAL",
    "classification": "MINOR",
    "department": "Quality Assurance"
})

print(f"Created deviation: {deviation.deviation_number}")
🤝 Support
For API integration support, contact info@masetra.com or call +256-760-927180.

Last updated: {{ site.time | date: '%B %d, %Y' }}