# Service API Documentation

## Overview
This document describes the internal REST API exposed by the HubSpot Deals ETL backend (Django). The service allows starting a scan, checking its status, and verifying system health.
Full interactive API documentation is available via Swagger/OpenAPI.

## Base URL
`/api/v1`

## Endpoints

### 1. Health Check
Checks if the API and database are reachable and functioning correctly.
- **URL**: `/health/`
- **Method**: `GET`
- **Authentication**: None required for basic health check.
- **Success Response**:
  - **Code**: `200 OK`
  - **Content**:
    ```json
    {
      "status": "healthy",
      "timestamp": "2026-08-29T10:00:00Z",
      "database": "connected"
    }
    ```

### 2. Start Scan
Triggers the ETL job to fetch deals from HubSpot.
- **URL**: `/scan/start/`
- **Method**: `POST`
- **Authentication**: Required (e.g., API Key or JWT)
- **Request Body**:
  ```json
  {
    "tenant_id": "tenant-123",
    "organization_id": "org-456",
    "filters": {
      "pipeline": "default"
    }
  }
  ```
- **Success Response**:
  - **Code**: `202 Accepted`
  - **Content**:
    ```json
    {
      "scan_id": "uuid-v4-1234-5678",
      "status": "pending",
      "message": "Scan job successfully queued."
    }
    ```
- **Error Responses**:
  - `400 Bad Request`: Invalid parameters.
  - `401 Unauthorized`: Missing or invalid authentication token.

### 3. Scan Status
Retrieves the current status and results summary of a specific scan.
- **URL**: `/scan/{scan_id}/status/`
- **Method**: `GET`
- **Authentication**: Required
- **Success Response**:
  - **Code**: `200 OK`
  - **Content**:
    ```json
    {
      "scan_id": "uuid-v4-1234-5678",
      "status": "completed",
      "records_extracted": 150,
      "started_at": "2026-08-29T10:00:00Z",
      "completed_at": "2026-08-29T10:05:00Z",
      "errors": []
    }
    ```
- **Error Responses**:
  - `404 Not Found`: Scan ID does not exist.

### 4. Interactive Swagger / OpenAPI Docs
- **URL**: `/swagger/` or `/api/schema/swagger-ui/`
- Details fully typed schemas, auth parameters, and testing console.
