# HubSpot Deals API Integration

## Overview
This document outlines the integration with the HubSpot CRM API v3 to extract deal records.

## API Endpoint
- **URL**: `https://api.hubapi.com/crm/v3/objects/deals`
- **Method**: `GET`
- **Documentation**: [HubSpot CRM API v3 - Deals](https://developers.hubspot.com/docs/api/crm/deals)

## Authentication
Authentication is performed using a Private App Access Token.
- **Header**: `Authorization: Bearer <YOUR_ACCESS_TOKEN>`

## Request Parameters
| Parameter | Type | Description |
|-----------|------|-------------|
| `limit` | integer | The maximum number of results to display per page (default: 10, max: 100). |
| `after` | string | The paging cursor token of the last successfully read resource. |
| `properties` | string | A comma-separated list of the properties to be returned in the response. |
| `archived` | boolean | Whether to return only results that have been archived. (default: false) |

## Pagination
The API uses cursor-based pagination. If more results are available, the response will include a `paging.next.after` string. This token should be passed as the `after` query parameter in the subsequent request.

## Rate Limiting
- **Limit**: 150 requests per 10 seconds.
- **Handling**: The integration must implement a retry strategy with exponential backoff and a circuit breaker pattern to prevent hitting the rate limits. Requests should pause gracefully if the limit is approaching.

## Error Handling
- **401 Unauthorized**: Invalid or missing access token.
- **429 Too Many Requests**: Rate limit exceeded. Service should pause and retry.
- **400 Bad Request**: Invalid parameters (e.g., malformed cursor).
- **5XX Server Errors**: HubSpot is experiencing issues. Retry with exponential backoff.

## Deal Properties Extracted
- `hs_object_id` (Deal ID)
- `amount` (Deal Amount)
- `dealstage` (Deal Stage)
- `dealname` (Deal Name)
- `closedate` (Close Date)
- `createdate` (Create Date)
- `hs_lastmodifieddate` (Last Modified Date)
- `description` (Description)
- `pipeline` (Pipeline ID)

## Example Request
```bash
curl -X GET \
  'https://api.hubapi.com/crm/v3/objects/deals?limit=100&properties=amount,dealstage,dealname,closedate,createdate,hs_lastmodifieddate,description,pipeline' \
  -H 'Authorization: Bearer YOUR_ACCESS_TOKEN'
```

## Example Response
```json
{
  "results": [
    {
      "id": "12345678",
      "properties": {
        "amount": "50000.00",
        "closedate": "2024-12-31T23:59:59.999Z",
        "createdate": "2024-01-01T00:00:00.000Z",
        "dealname": "Enterprise Deal",
        "dealstage": "appointmentscheduled",
        "description": "High value prospective deal.",
        "hs_lastmodifieddate": "2024-06-15T12:00:00.000Z",
        "pipeline": "default"
      },
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-06-15T12:00:00.000Z",
      "archived": false
    }
  ],
  "paging": {
    "next": {
      "after": "NTI1Cg%3D%3D",
      "link": "?after=NTI1Cg%3D%3D"
    }
  }
}
```
