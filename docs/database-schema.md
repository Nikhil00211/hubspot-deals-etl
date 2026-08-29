# Database Schema Design

## Overview
This document details the PostgreSQL database schema for the HubSpot Deals ETL backend. The application relies on a multi-tenant design, storing extracted deal data.

## Multi-Tenant Isolation Strategy
**Tenant ID Column (`_tenant_id`)**: Every table includes a `_tenant_id` column. Queries to access or modify data will be heavily scoped using this identifier at the application level (e.g., ORM filtering, row-level security if needed). This provides logical isolation within a shared database, balancing scalability with security. It allows handling multiple organizations using the same service instance without data leakage.

## Table: `hubspot_deals`

### Description
Stores deal records extracted from HubSpot CRM.

### Schema (CREATE TABLE SQL)

```sql
CREATE TABLE hubspot_deals (
    deal_id VARCHAR(255) PRIMARY KEY,
    deal_name VARCHAR(512),
    amount NUMERIC(15, 2),
    deal_stage VARCHAR(100),
    description TEXT,
    pipeline_id VARCHAR(255),
    
    -- HubSpot Timestamps
    hs_created_at TIMESTAMP WITH TIME ZONE,
    hs_updated_at TIMESTAMP WITH TIME ZONE,
    hs_closed_at TIMESTAMP WITH TIME ZONE,
    
    -- Extraction Metadata
    _extracted_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    _scan_id VARCHAR(255) NOT NULL,
    _tenant_id VARCHAR(255) NOT NULL,
    
    -- Constraints & Indexes
    CONSTRAINT chk_amount_positive CHECK (amount >= 0)
);

-- Indexes for performance
CREATE INDEX idx_hubspot_deals_tenant_id ON hubspot_deals(_tenant_id);
CREATE INDEX idx_hubspot_deals_scan_id ON hubspot_deals(_scan_id);
CREATE INDEX idx_hubspot_deals_deal_stage ON hubspot_deals(deal_stage);
CREATE INDEX idx_hubspot_deals_hs_updated_at ON hubspot_deals(hs_updated_at);
```

### Columns Explanation
- `deal_id`: The unique `hs_object_id` from HubSpot, acting as the Primary Key.
- `deal_name`: The name of the deal (`dealname`).
- `amount`: Deal amount, stored as a numeric/decimal to ensure precision without floating-point errors.
- `deal_stage`: The current pipeline stage of the deal.
- `description`: A text field for the deal's description.
- `pipeline_id`: The identifier for the pipeline the deal belongs to.
- `hs_created_at`, `hs_updated_at`, `hs_closed_at`: Deal lifecycle timestamps from HubSpot, stored with timezone information.
- `_extracted_at`: The timestamp when the record was ingested into our database.
- `_scan_id`: An identifier tying the record to a specific ETL sync/scan job.
- `_tenant_id`: The organizational tenant identifier for logical isolation.
