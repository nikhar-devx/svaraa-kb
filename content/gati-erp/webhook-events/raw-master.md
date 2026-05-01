---
title: Raw Master Webhook Events
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# Raw Master Webhook Events

## Event Name
`raw_master`

## Overview
Raw Master webhook events handle the synchronization of raw material information from the ERP system. Raw materials include precious metals, diamonds, gemstones, and other materials used in jewelry manufacturing.

## Event Payload Structure

```typescript
{
  operation: "create" | "update" | "delete",
  data: RawMasterData[]
}
```

## Purpose

Raw Master events:
- **Store Material Data**: Keep material reference information
- **Update Variant Options**: Sync material names to product options
- **Maintain Consistency**: Ensure material names are uniform across products
- **Queue Sync Jobs**: Trigger background updates for affected variants

## Raw Material Types

### Metals
- Gold (24K, 22K, 18K, 14K, etc.)
- Silver
- Platinum
- Palladium

### Diamonds
- Natural Diamonds
- Lab-Grown Diamonds
- Diamond melee

### Gemstones
- Ruby, Sapphire, Emerald
- Pearl
- Semi-precious stones

### Other Materials
- Cubic Zirconia (CZ)
- Synthetic stones
- Alloys and solders

## Operations

### 1. CREATE Operation

**When**: New raw material added to ERP

**Flow**:
```
ERP Raw Master Created
  └── Webhook Sent (raw_master/create)
        └── Workflow Processes
              ├── Create Raw Master Record
              │     ├── Store Raw Data
              │     └── Set Initial Title
              └── No Immediate Variant Impact
```

**Creates**:
- **RawMaster** record in plugin database
- No immediate Medusa entity created
- Used for reference in product details

**Data Stored**:
- `raw_no`: Material number
- `raw_code`: Unique code (e.g., "GOLD-22K")
- `raw_name`: Material name
- `raw_alias_name`: Display name
- `raw_mit_no`: Material type number
- `raw_mit_name`: Material type name
- `material_group`: Category (METAL, DIAMOND, etc.)

### 2. UPDATE Operation

**When**: Existing raw material modified in ERP

**Flow**:
```
ERP Raw Master Updated
  └── Webhook Sent (raw_master/update)
        └── Workflow Processes
              ├── Update Raw Master Record
              │     ├── Update Name/Alias
              │     └── Update Title
              ├── Check Title Changed
              │     └── If Changed:
              │           ├── Create Queue Entry
              │           │     └── VariantOptionSyncQueue
              │           └── Trigger Background Job
              │                 └── Update Variant Options
              └── Log Update
```

**Updates**:
- **RawMaster**: Name, alias, title fields
- **VariantOptionSyncQueue**: Entry created if title changed
- **Variant Options**: Updated via background job

**Important**:
- Title changes trigger variant option sync
- Background job updates all affected variants
- May take time for large product catalogs

### 3. DELETE Operation

**When**: Raw material deleted from ERP

**Flow**:
```
ERP Raw Master Deleted
  └── Webhook Sent (raw_master/delete)
        └── Delete Raw Master Record
              └── No Variant Impact (orphan references)
```

**Deletes**:
- **RawMaster** record
- Does not delete variant options
- Product references become stale

## Variant Option Sync Queue

### When Queue Entry Created
When `raw_name`, `raw_alias_name`, or `title` changes:

```typescript
{
  master_type: "RawMst",
  master_code: "GOLD-22K",
  master_id: "raw_123456",
  old_title: "22K Gold",
  new_title: "22 Karat Gold",
  status: "pending"
}
```

### Background Processing
```
Queue Entry Created
  └── Background Job Picks Up
        ├── Find All Affected Variants
        ├── Update Variant Option Titles
        ├── Sync to OpenSearch
        └── Mark Queue Entry Complete
```

## Usage in Products

### Style Details Line (ExtendedProduct)
Raw materials are referenced in product specifications:

```typescript
{
  style_id: 10001,
  item_name: "22K Gold",
  raw_no: 1001,
  raw_code: "GOLD-22K",
  raw_type: "Metal",
  // ... other fields
}
```

### Variant Options
Raw master titles become variant options:
- Option Name: "Metal"
- Option Value: "22K Gold" (from `raw_name` or `title`)

## Material Group Organization

Materials are organized by groups:

| Group | Materials |
|-------|-----------|
| METAL | Gold, Silver, Platinum |
| DIAMOND | All diamond types |
| GEMSTONE | Ruby, Sapphire, Emerald |
| SYNTHETIC | Lab-created stones |
| OTHER | Miscellaneous materials |

## Error Scenarios

### Title Update Fails
```
Raw Master Title Changed
  └── Queue Entry Created
        └── Background Job Fails
              └── Mark Failed in Queue
                    └── Log Error for Retry
```

### Orphaned References
```
Raw Master Deleted
  └── Product Still References raw_no
        └── Stale Data in Products
              └── May Cause Display Issues
```

## Event Sequence

### Material Addition Flow
```
1. raw_master/create
     └── Material Stored
     ↓
2. Product Created with Material
     └── References Raw Master
     ↓
3. Variant Option Created
     └── Uses Raw Master Name
```

### Material Update Flow
```
1. raw_master/update (title changed)
     └── Raw Master Updated
     ↓
2. Queue Entry Created
     └── VariantOptionSyncQueue
     ↓
3. Background Job Processes
     └── Updates All Variant Options
     ↓
4. OpenSearch Sync
     └── Search Index Updated
```

## Best Practices

1. **Consistent Naming**: Use clear, standard names
2. **Update Carefully**: Title changes affect many products
3. **Group Materials**: Organize by material_group
4. **Monitor Queue**: Check sync queue for failures
5. **Avoid Deletion**: Mark inactive instead of delete

## Related Events

- `quality_master`: Diamond/gem quality
- `tone_master`: Color tones
- `shape_master`: Stone shapes
- `item-size-master`: Size options
- `variant-option-sync-queue`: Background sync

## Performance Considerations

- Title updates are batched
- Background jobs run asynchronously
- Queue prevents duplicate updates
- Large catalogs may take time to sync

## Debugging

### Check Raw Master
```typescript
const raw = await rawMasterService.listRawMasters({
  raw_code: "GOLD-22K"
});
```

### Check Sync Queue
```typescript
const queue = await syncQueueService.listVariantOptionSyncQueues({
  master_type: "RawMst",
  status: "pending"
});
```

### Verify Variant Options
```typescript
const { data: variants } = await query.graph({
  entity: "product_variant",
  fields: ["*", "options.*"],
  filters: { /* relevant filters */ }
});
```
