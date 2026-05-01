---
title: ERP Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# ERP Module

## Overview

The ERP module manages ERP system configuration and integration settings. It stores connection details and credentials required to communicate with the Gati ERP system.

## Model: Erp

### Database Table
`erp`

### Fields

| Field | Type | Required | Indexed | Description |
|-------|------|----------|---------|-------------|
| `id` | ID | Yes | Yes (PK) | Primary key - auto-generated |
| `erp_id` | Text | Yes | Yes | ERP system identifier |
| `erp_name` | Text | Yes | No | ERP system name |

## Purpose

The ERP module serves as:
- Configuration storage for ERP connections
- Multi-ERP support (future capability)
- Integration metadata storage
- Webhook endpoint configuration reference

## Configuration

Typical ERP configuration includes:

```typescript
{
  erp_id: "GATI-PROD",
  erp_name: "Gati ERP Production"
}
```

## Usage

### Service Methods

```typescript
// Get ERP service
const erpService = container.resolve(ERP_MODULE);

// Create ERP configuration
const erp = await erpService.createErps([{
  erp_id: "GATI-PROD",
  erp_name: "Gati ERP Production"
}]);

// Update
await erpService.updateErps([{
  id: erp[0].id,
  erp_name: "Gati ERP Production System"
}]);

// List ERP configurations
const erps = await erpService.listErps();

// Retrieve specific ERP
const erpConfig = await erpService.retrieveErp(erp[0].id);
```

## Integration Points

The ERP configuration is used by:
- Webhook handlers to validate source
- Batch jobs to identify target ERP
- Event processors for routing
- API clients for authentication

## Future Enhancements

Planned additions to the ERP module:
- API endpoint URLs
- Authentication credentials (encrypted)
- Sync schedule configuration
- Last sync timestamp
- Error logging and retry configuration
- Webhook secret keys

## Multi-ERP Support

The module structure supports multiple ERP systems:

```typescript
// Primary ERP
{ erp_id: "GATI-PROD", erp_name: "Gati ERP Production" }

// Secondary ERP (future)
{ erp_id: "GATI-TEST", erp_name: "Gati ERP Testing" }

// Legacy ERP (future)
{ erp_id: "GATI-LEGACY", erp_name: "Gati ERP Legacy" }
```

## Webhook Validation

ERP records can be used to validate incoming webhooks:

```typescript
// Validate webhook source
const validateWebhook = async (erpId: string, webhookData: any) => {
  const erpService = container.resolve(ERP_MODULE);
  const erp = await erpService.listErps({ erp_id: erpId });
  
  if (erp.length === 0) {
    throw new Error("Invalid ERP source");
  }
  
  // Process webhook
};
```
