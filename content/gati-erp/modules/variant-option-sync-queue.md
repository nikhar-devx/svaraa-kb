---
title: Variant Option Sync Queue Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# Variant Option Sync Queue Module

## Overview

The Variant Option Sync Queue module manages background synchronization of variant option values when master data changes. This ensures that updates to Raw Master, Quality Master, Tone Master, Shape Master, and Item Size Master are propagated to all affected product variants.

## Model: VariantOptionSyncQueue

### Database Table
`variant_option_sync_queue`

### Fields

| Field | Type | Required | Indexed | Description |
|-------|------|----------|---------|-------------|
| `id` | ID | Yes | Yes (PK) | Primary key - auto-generated |
| `master_type` | Enum | Yes | Yes | Type of master (RawMst, QualityMst, etc.) |
| `master_code` | Text | Yes | Yes | Code of the updated master |
| `master_id` | Text | Yes | Yes | ID of the updated master record |
| `old_title` | Text | Yes | No | Previous title/value |
| `new_title` | Text | Yes | No | New title/value |
| `status` | Enum | Yes | Yes | Queue entry status |
| `processed_at` | DateTime | No | No | Processing timestamp |
| `error_message` | Text | No | No | Error details if failed |
| `affected_variant_count` | Number | Yes | No | Number of variants updated |
| `updated_by` | Text | No | No | User/system that triggered update |

## Master Types

| Type | Description |
|------|-------------|
| `RawMst` | Raw material master |
| `QualityMst` | Quality/clarity master |
| `ToneMst` | Tone/color master |
| `ItemSizeMst` | Item size master |
| `ShapeMst` | Shape master |

## Status Values

| Status | Description |
|--------|-------------|
| `pending` | Entry created, awaiting processing |
| `processing` | Currently being processed |
| `completed` | All variants updated successfully |
| `failed` | Processing failed |

## Purpose

The sync queue provides:
- **Asynchronous Updates**: Master updates don't block requests
- **Batch Processing**: Update multiple variants efficiently
- **Error Handling**: Failed updates can be retried
- **Audit Trail**: Track what changed and when
- **Performance**: Avoid processing duplicates

## Queue Processing Flow

```
Master Data Updated (e.g., Raw Master title)
  └── Queue Entry Created (status: pending)
        └── Background Job Processes Queue
              ├── Find All Affected Variants
              ├── Update Variant Options
              ├── Sync to OpenSearch
              └── Update Queue Entry (status: completed)
```

## Usage

### Service Methods

```typescript
// Get sync queue service
const syncQueueService = container.resolve(VARIANT_OPTION_SYNC_QUEUE_MODULE);

// Create queue entry (triggered by master update)
const queueEntry = await syncQueueService.createVariantOptionSyncQueues([{
  master_type: "RawMst",
  master_code: "GOLD-22K",
  master_id: "raw_123456",
  old_title: "22K Gold",
  new_title: "22 Karat Gold",
  status: "pending"
}]);

// Update status
await syncQueueService.updateVariantOptionSyncQueues([{
  id: queueEntry[0].id,
  status: "processing"
}]);

// Mark completed
await syncQueueService.updateVariantOptionSyncQueues([{
  id: queueEntry[0].id,
  status: "completed",
  processed_at: new Date(),
  affected_variant_count: 150
}]);

// List pending entries
const pending = await syncQueueService.listVariantOptionSyncQueues({
  status: "pending"
});
```

## Batch Processing Job

The module includes a batch job that processes the queue:

```typescript
// Process all pending entries
const processQueue = async () => {
  const syncQueueService = container.resolve(VARIANT_OPTION_SYNC_QUEUE_MODULE);
  
  const pendingEntries = await syncQueueService.listVariantOptionSyncQueues({
    status: "pending"
  });
  
  for (const entry of pendingEntries) {
    try {
      // Update status to processing
      await syncQueueService.updateVariantOptionSyncQueues([{
        id: entry.id,
        status: "processing"
      }]);
      
      // Find and update affected variants
      const count = await updateVariantOptions(entry);
      
      // Mark completed
      await syncQueueService.updateVariantOptionSyncQueues([{
        id: entry.id,
        status: "completed",
        processed_at: new Date(),
        affected_variant_count: count
      }]);
    } catch (error) {
      // Mark failed
      await syncQueueService.updateVariantOptionSyncQueues([{
        id: entry.id,
        status: "failed",
        error_message: error.message
      }]];
    }
  }
};
```

## Trigger Conditions

Queue entries are created when:
1. **Raw Master**: `title`, `raw_name`, or `raw_alias_name` changes
2. **Quality Master**: `title`, `qly_name`, or `qly_alias_name` changes
3. **Tone Master**: `title`, `tone_name`, or `tone_alias_name` changes
4. **Shape Master**: `title`, `shape_name`, or `shape_alias_name` changes
5. **Item Size Master**: `title`, `item_size_name`, or `item_size_alias_name` changes

## Error Handling

Failed entries:
- Store error message in `error_message`
- Can be retried by resetting status to `pending`
- Should be monitored and alerted

## Performance Considerations

- Process queue in batches (e.g., 100 entries at a time)
- Update variants in bulk where possible
- Emit single sync event to OpenSearch per batch
- Monitor `affected_variant_count` for anomalies
