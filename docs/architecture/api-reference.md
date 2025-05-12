# Application Packaging Automation System (APAS) API Reference

## API Overview

The APAS system exposes a RESTful API that enables interaction with the packaging system, management of packages, and monitoring of the packaging process. The API is organized around core resources and follows standard HTTP methods and status codes.

- **Base URL:** `/api/v1`
- **Format:** All requests and responses use JSON
- **Authentication:** JWT-based authentication for all endpoints
- **Error Handling:** Consistent error response format with appropriate HTTP status codes

## OpenAPI Specification

The complete API is documented using OpenAPI/Swagger and available at `/api/v1/docs` when the server is running.

## Core Resources

### Packages

Resources related to application packages, including creation, monitoring, and management.

#### `GET /packages`

Retrieve a list of packages with pagination, filtering, and sorting.

- **Query Parameters:**
  - `page`: Page number (default: 1)
  - `limit`: Items per page (default: 20)
  - `status`: Filter by status (e.g., `draft`, `completed`)
  - `search`: Search term for package name or publisher
  - `sort`: Sort field (e.g., `created_at`, `name`)
  - `order`: Sort order (`asc` or `desc`, default: `desc`)

- **Response (200 OK):**
  ```json
  {
    "items": [
      {
        "id": "123e4567-e89b-12d3-a456-426614174000",
        "name": "Example Application",
        "version": "1.0.0",
        "publisher": "Example Publisher",
        "status": "completed",
        "created_at": "2025-04-01T12:00:00Z",
        "updated_at": "2025-04-01T12:30:00Z"
      }
    ],
    "total": 100,
    "page": 1,
    "limit": 20,
    "pages": 5
  }
  ```

#### `POST /packages`

Create a new package.

- **Request Body:**
  ```json
  {
    "name": "Example Application",
    "version": "1.0.0",
    "publisher": "Example Publisher",
    "description": "An example application for testing."
  }
  ```

- **Response (201 Created):**
  ```json
  {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "name": "Example Application",
    "version": "1.0.0",
    "publisher": "Example Publisher",
    "description": "An example application for testing.",
    "status": "draft",
    "created_at": "2025-04-01T12:00:00Z",
    "updated_at": "2025-04-01T12:00:00Z"
  }
  ```

#### `GET /packages/{package_id}`

Retrieve a specific package by ID.

- **Path Parameters:**
  - `package_id`: Package ID

- **Response (200 OK):**
  ```json
  {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "name": "Example Application",
    "version": "1.0.0",
    "publisher": "Example Publisher",
    "description": "An example application for testing.",
    "status": "completed",
    "created_at": "2025-04-01T12:00:00Z",
    "updated_at": "2025-04-01T12:30:00Z",
    "installer": {
      "id": "123e4567-e89b-12d3-a456-426614174001",
      "filename": "example.msi",
      "file_type": "msi",
      "file_size": 1024000,
      "architecture": "x64"
    },
    "psadt_script": {
      "id": "123e4567-e89b-12d3-a456-426614174002",
      "template_version": "3.9.3"
    },
    "wdac_policy": {
      "id": "123e4567-e89b-12d3-a456-426614174003",
      "policy_type": "audit"
    },
    "documentation": {
      "id": "123e4567-e89b-12d3-a456-426614174004",
      "version": 1
    }
  }
  ```

#### `PATCH /packages/{package_id}`

Update a specific package.

- **Path Parameters:**
  - `package_id`: Package ID

- **Request Body:**
  ```json
  {
    "name": "Updated Application Name",
    "description": "Updated description."
  }
  ```

- **Response (200 OK):**
  ```json
  {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "name": "Updated Application Name",
    "version": "1.0.0",
    "publisher": "Example Publisher",
    "description": "Updated description.",
    "status": "draft",
    "created_at": "2025-04-01T12:00:00Z",
    "updated_at": "2025-04-01T12:05:00Z"
  }
  ```

#### `DELETE /packages/{package_id}`

Delete a specific package.

- **Path Parameters:**
  - `package_id`: Package ID

- **Response (204 No Content)**

### Installers

Resources related to application installers, including upload, analysis, and metadata.

#### `POST /packages/{package_id}/installer`

Upload and analyze an installer file for a specific package.

- **Path Parameters:**
  - `package_id`: Package ID

- **Request Body:**
  Multipart form data with:
  - `file`: The installer file
  - `options`: (Optional) JSON string with analysis options
    ```json
    {
      "detailed_analysis": true,
      "external_research": true,
      "ai_assistance": true
    }
    ```

- **Response (202 Accepted):**
  ```json
  {
    "id": "123e4567-e89b-12d3-a456-426614174001",
    "package_id": "123e4567-e89b-12d3-a456-426614174000",
    "filename": "example.msi",
    "file_size": 1024000,
    "status": "analyzing",
    "created_at": "2025-04-01T12:10:00Z",
    "updated_at": "2025-04-01T12:10:00Z"
  }
  ```

#### `GET /packages/{package_id}/installer`

Get installer information for a specific package.

- **Path Parameters:**
  - `package_id`: Package ID

- **Response (200 OK):**
  ```json
  {
    "id": "123e4567-e89b-12d3-a456-426614174001",
    "package_id": "123e4567-e89b-12d3-a456-426614174000",
    "filename": "example.msi",
    "file_type": "msi",
    "file_size": 1024000,
    "file_hash": "abcdef1234567890",
    "architecture": "x64",
    "framework": "Advanced Installer",
    "metadata": {
      "product_name": "Example Application",
      "product_version": "1.0.0",
      "product_publisher": "Example Publisher",
      "product_language": "en-US",
      "product_code": "{12345678-1234-1234-1234-123456789012}",
      "requires_elevation": true
    },
    "silent_switches": "/qn /norestart",
    "analysis_results": {
      "confidence_score": 0.95,
      "detected_requirements": [
        ".NET Framework 4.8",
        "Windows 10 or higher"
      ],
      "potential_issues": [],
      "recommendations": [
        "Use /qn /norestart for silent installation"
      ],
      "external_research": [
        {
          "source": "Vendor Documentation",
          "url": "https://example.com/docs",
          "relevance_score": 0.9,
          "information": "The example application supports silent installation."
        }
      ]
    },
    "created_at": "2025-04-01T12:10:00Z",
    "updated_at": "2025-04-01T12:15:00Z"
  }
  ```

#### `GET /packages/{package_id}/installer/download`

Download the installer file for a specific package.

- **Path Parameters:**
  - `package_id`: Package ID

- **Response (200 OK):**
  Binary file download

### PSADT Scripts

Resources related to PowerShell AppDeploy Toolkit scripts, including generation, testing, and management.

#### `POST /packages/{package_id}/psadt`

Generate a PSADT script for a specific package.

- **Path Parameters:**
  - `package_id`: Package ID

- **Request Body:**
  ```json
  {
    "template_id": "123e4567-e89b-12d3-a456-426614174005",
    "options": {
      "include_pre_installation": true,
      "include_post_installation": true,
      "custom_parameters": {
        "param1": "value1",
        "param2": "value2"
      }
    }
  }
  ```

- **Response (202 Accepted):**
  ```json
  {
    "id": "123e4567-e89b-12d3-a456-426614174002",
    "package_id": "123e4567-e89b-12d3-a456-426614174000",
    "template_id": "123e4567-e89b-12d3-a456-426614174005",
    "status": "generating",
    "created_at": "2025-04-01T12:20:00Z",
    "updated_at": "2025-04-01T12:20:00Z"
  }
  ```

#### `GET /packages/{package_id}/psadt`

Get PSADT script information for a specific package.

- **Path Parameters:**
  - `package_id`: Package ID

- **Response (200 OK):**
  ```json
  {
    "id": "123e4567-e89b-12d3-a456-426614174002",
    "package_id": "123e4567-e89b-12d3-a456-426614174000",
    "template_id": "123e4567-e89b-12d3-a456-426614174005",
    "template_version": "3.9.3",
    "pre_installation": [
      "Stop-Process -Name 'example' -Force -ErrorAction SilentlyContinue"
    ],
    "installation": [
      "Execute-MSI -Action Install -Path '$dirFiles\\example.msi' -Parameters '/qn /norestart'"
    ],
    "post_installation": [
      "Start-Process -FilePath 'example.exe'"
    ],
    "test_results": [
      {
        "id": "123e4567-e89b-12d3-a456-426614174006",
        "timestamp": "2025-04-01T12:25:00Z",
        "success": true,
        "logs": ["Installation completed successfully."],
        "execution_time": 120,
        "exit_code": 0,
        "issues": []
      }
    ],
    "created_at": "2025-04-01T12:20:00Z",
    "updated_at": "2025-04-01T12:25:00Z"
  }
  ```

#### `GET /packages/{package_id}/psadt/download`

Download the PSADT script for a specific package.

- **Path Parameters:**
  - `package_id`: Package ID

- **Response (200 OK):**
  ZIP file containing PSADT script and associated files

#### `POST /packages/{package_id}/psadt/test`

Test the PSADT script for a specific package.

- **Path Parameters:**
  - `package_id`: Package ID

- **Request Body:**
  ```json
  {
    "mode": "simulation" // or "actual" for actual installation
  }
  ```

- **Response (202 Accepted):**
  ```json
  {
    "test_id": "123e4567-e89b-12d3-a456-426614174006",
    "package_id": "123e4567-e89b-12d3-a456-426614174000",
    "mode": "simulation",
    "status": "running",
    "started_at": "2025-04-01T12:25:00Z"
  }
  ```

### WDAC Policies

Resources related to Windows Defender Application Control policies, including generation, testing, and management.

#### `POST /packages/{package_id}/wdac`

Generate a WDAC policy for a specific package.

- **Path Parameters:**
  - `package_id`: Package ID

- **Request Body:**
  ```json
  {
    "policy_type": "audit", // or "enforcement"
    "is_supplemental": false,
    "base_policy_guid": null, // required if is_supplemental is true
    "options": {
      "allow_publishers": true,
      "allow_file_paths": true,
      "allow_file_hashes": true
    }
  }
  ```

- **Response (202 Accepted):**
  ```json
  {
    "id": "123e4567-e89b-12d3-a456-426614174003",
    "package_id": "123e4567-e89b-12d3-a456-426614174000",
    "policy_type": "audit",
    "is_supplemental": false,
    "status": "generating",
    "created_at": "2025-04-01T12:30:00Z",
    "updated_at": "2025-04-01T12:30:00Z"
  }
  ```

#### `GET /packages/{package_id}/wdac`

Get WDAC policy information for a specific package.

- **Path Parameters:**
  - `package_id`: Package ID

- **Response (200 OK):**
  ```json
  {
    "id": "123e4567-e89b-12d3-a456-426614174003",
    "package_id": "123e4567-e89b-12d3-a456-426614174000",
    "policy_name": "Example Application Policy",
    "policy_type": "audit",
    "policy_guid": "12345678-1234-1234-1234-123456789012",
    "policy_version": 1,
    "is_supplemental": false,
    "base_policy_guid": null,
    "rules": [
      {
        "id": "123e4567-e89b-12d3-a456-426614174007",
        "rule_type": "publisher",
        "rule_level": "allow",
        "value": "O=Example Publisher, CN=Example Publisher Certificate",
        "description": "Allow publisher Example Publisher",
        "is_auto_generated": true
      }
    ],
    "signing_info": {
      "signed_files": 5,
      "unsigned_files": 0,
      "publishers": [
        {
          "name": "Example Publisher",
          "thumbprint": "1234567890abcdef1234567890abcdef12345678",
          "file_count": 5
        }
      ]
    },
    "test_results": [
      {
        "id": "123e4567-e89b-12d3-a456-426614174008",
        "timestamp": "2025-04-01T12:35:00Z",
        "mode": "audit",
        "success": true,
        "violations": [],
        "logs": ["No policy violations detected."]
      }
    ],
    "created_at": "2025-04-01T12:30:00Z",
    "updated_at": "2025-04-01T12:35:00Z"
  }
  ```

#### `GET /packages/{package_id}/wdac/download`

Download the WDAC policy for a specific package.

- **Path Parameters:**
  - `package_id`: Package ID

- **Response (200 OK):**
  XML file containing WDAC policy

#### `POST /packages/{package_id}/wdac/test`

Test the WDAC policy for a specific package.

- **Path Parameters:**
  - `package_id`: Package ID

- **Request Body:**
  ```json
  {
    "mode": "audit" // or "enforcement"
  }
  ```

- **Response (202 Accepted):**
  ```json
  {
    "test_id": "123e4567-e89b-12d3-a456-426614174008",
    "package_id": "123e4567-e89b-12d3-a456-426614174000",
    "mode": "audit",
    "status": "running",
    "started_at": "2025-04-01T12:35:00Z"
  }
  ```

### Documentation

Resources related to package documentation, including generation and management.

#### `POST /packages/{package_id}/documentation`

Generate documentation for a specific package.

- **Path Parameters:**
  - `package_id`: Package ID

- **Request Body:**
  ```json
  {
    "options": {
      "include_installation_steps": true,
      "include_troubleshooting": true,
      "include_screenshots": true
    }
  }
  ```

- **Response (202 Accepted):**
  ```json
  {
    "id": "123e4567-e89b-12d3-a456-426614174004",
    "package_id": "123e4567-e89b-12d3-a456-426614174000",
    "status": "generating",
    "created_at": "2025-04-01T12:40:00Z",
    "updated_at": "2025-04-01T12:40:00Z"
  }
  ```

#### `GET /packages/{package_id}/documentation`

Get documentation information for a specific package.

- **Path Parameters:**
  - `package_id`: Package ID

- **Response (200 OK):**
  ```json
  {
    "id": "123e4567-e89b-12d3-a456-426614174004",
    "package_id": "123e4567-e89b-12d3-a456-426614174000",
    "content_markdown": "# Example Application 1.0.0\n\n## Installation\n\nThe application can be installed silently using the following command:\n\n```\nExample.msi /qn /norestart\n```\n\n## Requirements\n\n- Windows 10 or higher\n- .NET Framework 4.8\n",
    "sections": [
      {
        "id": "123e4567-e89b-12d3-a456-426614174009",
        "title": "Overview",
        "content": "Example Application is a sample application for testing.",
        "order": 1,
        "parent_id": null
      },
      {
        "id": "123e4567-e89b-12d3-a456-426614174010",
        "title": "Installation",
        "content": "The application can be installed silently using the following command:\n\n```\nExample.msi /qn /norestart\n```",
        "order": 2,
        "parent_id": null
      },
      {
        "id": "123e4567-e89b-12d3-a456-426614174011",
        "title": "Requirements",
        "content": "- Windows 10 or higher\n- .NET Framework 4.8",
        "order": 3,
        "parent_id": null
      }
    ],
    "version": 1,
    "created_at": "2025-04-01T12:40:00Z",
    "updated_at": "2025-04-01T12:45:00Z"
  }
  ```

#### `GET /packages/{package_id}/documentation/download`

Download the documentation for a specific package in the specified format.

- **Path Parameters:**
  - `package_id`: Package ID

- **Query Parameters:**
  - `format`: Documentation format (`pdf`, `markdown`, `html`, default: `pdf`)

- **Response (200 OK):**
  Documentation file in the requested format

### Events

Resources related to system events, including event history and management.

#### `GET /events`

Retrieve a list of system events with pagination, filtering, and sorting.

- **Query Parameters:**
  - `page`: Page number (default: 1)
  - `limit`: Items per page (default: 20)
  - `event_type`: Filter by event type
  - `topic`: Filter by topic
  - `package_id`: Filter by package ID
  - `start_date`: Filter by start date
  - `end_date`: Filter by end date
  - `sort`: Sort field (default: `timestamp`)
  - `order`: Sort order (`asc` or `desc`, default: `desc`)

- **Response (200 OK):**
  ```json
  {
    "items": [
      {
        "id": "123e4567-e89b-12d3-a456-426614174012",
        "event_type": "package.created",
        "topic": "package",
        "source": "api",
        "correlation_id": "123e4567-e89b-12d3-a456-426614174013",
        "package_id": "123e4567-e89b-12d3-a456-426614174000",
        "timestamp": "2025-04-01T12:00:00Z",
        "processed": true,
        "processing_time": 50
      }
    ],
    "total": 100,
    "page": 1,
    "limit": 20,
    "pages": 5
  }
  ```

#### `GET /events/{event_id}`

Retrieve a specific event by ID.

- **Path Parameters:**
  - `event_id`: Event ID

- **Response (200 OK):**
  ```json
  {
    "id": "123e4567-e89b-12d3-a456-426614174012",
    "event_type": "package.created",
    "topic": "package",
    "payload": {
      "package_id": "123e4567-e89b-12d3-a456-426614174000",
      "name": "Example Application",
      "version": "1.0.0",
      "publisher": "Example Publisher"
    },
    "source": "api",
    "correlation_id": "123e4567-e89b-12d3-a456-426614174013",
    "package_id": "123e4567-e89b-12d3-a456-426614174000",
    "timestamp": "2025-04-01T12:00:00Z",
    "processed": true,
    "processing_time": 50
  }
  ```

### Templates

Resources related to PSADT templates, including management and customization.

#### `GET /templates`

Retrieve a list of PSADT templates with pagination, filtering, and sorting.

- **Query Parameters:**
  - `page`: Page number (default: 1)
  - `limit`: Items per page (default: 20)
  - `search`: Search term for template name
  - `sort`: Sort field (default: `created_at`)
  - `order`: Sort order (`asc` or `desc`, default: `desc`)

- **Response (200 OK):**
  ```json
  {
    "items": [
      {
        "id": "123e4567-e89b-12d3-a456-426614174005",
        "name": "Standard Template",
        "description": "Standard PSADT template for most applications.",
        "psadt_version": "3.9.3",
        "created_at": "2025-03-01T12:00:00Z",
        "updated_at": "2025-03-01T12:00:00Z"
      }
    ],
    "total": 5,
    "page": 1,
    "limit": 20,
    "pages": 1
  }
  ```

#### `POST /templates`

Create a new PSADT template.

- **Request Body:**
  ```json
  {
    "name": "Custom Template",
    "description": "Custom PSADT template for specific applications.",
    "psadt_version": "3.9.3",
    "based_on_id": "123e4567-e89b-12d3-a456-426614174005",
    "customizations": {
      "pre_installation": ["# Custom pre-installation commands"],
      "post_installation": ["# Custom post-installation commands"]
    }
  }
  ```

- **Response (201 Created):**
  ```json
  {
    "id": "123e4567-e89b-12d3-a456-426614174014",
    "name": "Custom Template",
    "description": "Custom PSADT template for specific applications.",
    "psadt_version": "3.9.3",
    "based_on_id": "123e4567-e89b-12d3-a456-426614174005",
    "created_at": "2025-04-01T13:00:00Z",
    "updated_at": "2025-04-01T13:00:00Z"
  }
  ```

#### `GET /templates/{template_id}`

Retrieve a specific template by ID.

- **Path Parameters:**
  - `template_id`: Template ID

- **Response (200 OK):**
  ```json
  {
    "id": "123e4567-e89b-12d3-a456-426614174005",
    "name": "Standard Template",
    "description": "Standard PSADT template for most applications.",
    "psadt_version": "3.9.3",
    "content": "# Deploy-Application.ps1 template\n...",
    "customizations": {},
    "created_at": "2025-03-01T12:00:00Z",
    "updated_at": "2025-03-01T12:00:00Z"
  }
  ```

### Knowledge Base

Resources related to the knowledge base, including retrieval and management.

#### `GET /knowledge`

Retrieve knowledge items from the knowledge base with semantic search.

- **Query Parameters:**
  - `query`: Search query
  - `content_type`: Filter by content type
  - `category`: Filter by category
  - `tags`: Filter by tags (comma-separated)
  - `limit`: Maximum number of results (default: 10)

- **Response (200 OK):**
  ```json
  {
    "items": [
      {
        "id": "123e4567-e89b-12d3-a456-426614174015",
        "title": "Silent Installation for Example Application",
        "content": "Example Application supports silent installation with the /qn /norestart parameters.",
        "content_type": "installer_pattern",
        "category": "msi",
        "tags": ["silent", "msi", "example"],
        "relevance_score": 0.95,
        "created_at": "2025-03-15T12:00:00Z",
        "updated_at": "2025-03-15T12:00:00Z"
      }
    ],
    "total": 100
  }
  ```

#### `POST /knowledge`

Add a new knowledge item to the knowledge base.

- **Request Body:**
  ```json
  {
    "title": "Custom Installation Process",
    "content": "This is a custom installation process for a complex application.",
    "content_type": "best_practice",
    "category": "complex_installer",
    "tags": ["custom", "complex", "installation"],
    "source_package_id": "123e4567-e89b-12d3-a456-426614174000",
    "source_url": "https://example.com/docs"
  }
  ```

- **Response (201 Created):**
  ```json
  {
    "id": "123e4567-e89b-12d3-a456-426614174016",
    "title": "Custom Installation Process",
    "content": "This is a custom installation process for a complex application.",
    "content_type": "best_practice",
    "category": "complex_installer",
    "tags": ["custom", "complex", "installation"],
    "source_package_id": "123e4567-e89b-12d3-a456-426614174000",
    "source_url": "https://example.com/docs",
    "relevance_score": 0,
    "created_at": "2025-04-01T14:00:00Z",
    "updated_at": "2025-04-01T14:00:00Z"
  }
  ```

## Error Responses

All API endpoints use a consistent error response format:

```json
{
  "error": {
    "code": "not_found",
    "message": "Package with ID '123e4567-e89b-12d3-a456-426614174000' not found.",
    "details": {
      "resource": "package",
      "id": "123e4567-e89b-12d3-a456-426614174000"
    }
  }
}
```

### Common Error Codes

- `bad_request`: Request body or parameters are invalid.
- `unauthorized`: Authentication is required or invalid.
- `forbidden`: User lacks permission for the requested operation.
- `not_found`: Requested resource does not exist.
- `conflict`: Request conflicts with current state.
- `unprocessable_entity`: Request is valid but cannot be processed.
- `internal_server_error`: Unexpected server error.

## Rate Limiting

API requests are limited to 100 requests per minute per user. Rate limit information is included in response headers:

- `X-RateLimit-Limit`: Maximum requests per minute.
- `X-RateLimit-Remaining`: Remaining requests in the current window.
- `X-RateLimit-Reset`: Time (in seconds) until the rate limit window resets.

When the rate limit is exceeded, requests will receive a `429 Too Many Requests` response.

## Webhooks

The APAS system supports webhooks for event notifications. Webhooks can be configured to receive notifications for specific event types.

### Configuring Webhooks

```json
POST /webhooks

{
  "url": "https://example.com/webhook",
  "events": ["package.created", "package.completed", "package.failed"],
  "secret": "webhook_secret"
}
```

### Webhook Payload

```json
{
  "event_type": "package.completed",
  "timestamp": "2025-04-01T12:30:00Z",
  "package_id": "123e4567-e89b-12d3-a456-426614174000",
  "data": {
    "name": "Example Application",
    "version": "1.0.0",
    "publisher": "Example Publisher",
    "status": "completed"
  }
}
```

## Change Log

| Change        | Date       | Version | Description                 | Author         |
| ------------- | ---------- | ------- | --------------------------- | -------------- |
| Initial draft | 2025-05-08 | 0.1     | Initial API documentation   | Architect Agent |
