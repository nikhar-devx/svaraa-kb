---
title: ERP Event Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# ERP Event Module

## Overview

The ERP Event module logs and tracks all ERP webhook events received by the system. It provides an audit trail for data synchronization operations and helps monitor integration health.

## Model: ErpEvent

### Database Table
`erp_event`

### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | ID | Yes | Primary key - auto-generated |
| `datafor` | Text | Yes | Module/entity type (e.g., "party_master") |
| `operation` | Text | Yes | Operation type (create, update, delete) |
| `data` | Array | Yes | Event payload data |
| `sync_completed_at` | DateTime | No | Timestamp when sync completed |
| `status` | Enum | No | Event status |

## Status Values

| Status | Description |
|--------|-------------|
| `pending` | Event received, awaiting processing |
| `in-progress` | Event is being processed |
| `completed` | Event processed successfully |
| `failed` | Event processing failed |

## Purpose

The ERP Event module provides:
- **Audit Trail**: Log of all ERP webhook events
- **Debugging**: Track event processing status
- **Monitoring**: Identify failed or stuck events
- **Replay Capability**: Re-process failed events
- **Analytics**: Integration performance metrics

## Event Structure

Example event record:

```json
{
  "id": "erp_evt_123456789",
  "datafor": "party_master",
  "operation": "create",
  "data": [
    {
      "id": "pm_001",
      "party_no": "12345",
      "firm_name": "ABC Jewelers"
    }
  ],
  "sync_completed_at": null,
  "status": "pending"
}
```

## Usage

### Service Methods

```typescript
// Get ERP event service
const erpEventService = container.resolve(ERP_EVENT_MODULE);

// Create event log
const event = await erpEventService.createErpEvents([{
  datafor: "party_master",
  operation: "create",
  data: [/* webhook payload */],
  status: "pending"
}]);

// Update event status
await erpEventService.updateErpEvents([{
  id: event[0].id,
  status: "completed",
  sync_completed_at: new Date()
}]);

// List pending events
const pendingEvents = await erpEventService.listErpEvents({
  status: "pending"
});

// List failed events for retry
const failedEvents = await erpEventService.listErpEvents({
  status: "failed"
});
```

## Event Lifecycle

```
Webhook Received
  └── ErpEvent Created (status: pending)
        └── Subscriber Processes Event
              ├── Success → ErpEvent Updated (status: completed)
              └── Failure → ErpEvent Updated (status: failed)
                    └── Retry Logic (if configured)
                          └── Success/Permanent Failure
```

## Monitoring Queries

### Count Events by Status
```typescript
const stats = await erpEventService.listAndCountErpEvents({
  groupBy: "status"
});
```

### Recent Failed Events
```typescript
const recentFailures = await erpEventService.listErpEvents({
  status: "failed",
  created_at: { $gte: new Date(Date.now() - 24 * 60 * 60 * 1000) }
});
```

### Events by Module
```typescript
const moduleStats = await erpEventService.listErpEvents({
  datafor: "party_master"
});
```

## Integration with Batch Jobs

The ERP Event module can be used by batch jobs to:
1. Process pending events in bulk
2. Retry failed events
3. Clean up old completed events
4. Generate integration reports

## Retention Policy

Recommended data retention:
- **Pending/In-progress**: Keep until processed
- **Completed**: 30-90 days
- **Failed**: 90-180 days (for debugging)

Cleanup job example:
```typescript
// Delete events older than 90 days
const cutoffDate = new Date(Date.now() - 90 * 24 * 60 * 60 * 1000);
const oldEvents = await erpEventService.listErpEvents({
  status: "completed",
  created_at: { $lt: cutoffDate }
});
await erpEventService.deleteErpEvents(oldEvents.map(e => e.id));
```
