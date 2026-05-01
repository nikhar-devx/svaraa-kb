---
title: Item Size Master Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# Item Size Master Module

## Overview

The Item Size Master module manages size options for jewelry items synchronized from the ERP system. Sizes are applicable to rings, bracelets, necklaces, and other size-dependent jewelry pieces.

## Model: ItemSizeMaster

### Database Table
`item_size_master`

### Fields

| Field | Type | Required | Indexed | Description |
|-------|------|----------|---------|-------------|
| `id` | ID | Yes | Yes (PK) | Primary key - auto-generated |
| `title` | Text | No | No | Display title for variant options |
| `item_size_id` | Number | Yes | Yes | Size ID from ERP |
| `item_size_code` | Text | Yes | Yes | Unique size code |
| `item_size_name` | Text | Yes | No | Size name |
| `item_size_alias_name` | Text | Yes | No | Alternative/display name |
| `grp_nos` | Array | Yes | No | Associated category group numbers |

## Size Types

### Ring Sizes
Standard ring sizes (US/India):
- Size 5 to 15 (including half sizes)
- Examples: 5, 5.5, 6, 6.5, 7, etc.

### Bracelet Sizes
- Small (6-6.5 inches)
- Medium (7-7.5 inches)
- Large (8-8.5 inches)
- Extra Large (9+ inches)

### Necklace Lengths
- Choker (14-16 inches)
- Princess (17-19 inches)
- Matinee (20-24 inches)
- Opera (28-34 inches)
- Rope (45+ inches)

### Bangle Sizes
- Inner diameter measurements
- Circumference measurements
- Standard sizes: 2-2, 2-4, 2-6, 2-8, etc.

## Relationships

- **Referenced by** ExtendedProduct (via `item_size_id`)
- **Referenced by** ExtendedVariant (via `item_size_id`)
- **Many-to-Many** with Category Groups (via `grp_nos`)

## Category Group Associations

The `grp_nos` array contains category group numbers where this size applies:

```json
{
  "item_size_code": "RING-7",
  "item_size_name": "Ring Size 7",
  "grp_nos": [10, 11, 12]  // Applies to multiple category groups
}
```

This allows sizes to be shared across different product categories.

## Medusa Integration

Item Size Master data is used to:
1. Create/update variant option values in Medusa
2. Define product size options
3. Filter sizes by category group
4. Synchronize size information across products

### Variant Option Sync

When an Item Size Master title is updated:
1. Entry added to `VariantOptionSyncQueue`
2. Background job processes the queue
3. Updates all affected variant options in Medusa

## Usage

### Service Methods

```typescript
// Get item size master service
const itemSizeMasterService = container.resolve(ITEM_SIZE_MASTER_MODULE);

// Create item size master
const itemSize = await itemSizeMasterService.createItemSizeMasters([{
  item_size_id: 5001,
  item_size_code: "RING-7",
  item_size_name: "Ring Size 7",
  item_size_alias_name: "Size 7 (US)",
  grp_nos: [10, 11, 12]  // Rings, Bands, Engagement categories
}]);

// Update (triggers variant option sync)
await itemSizeMasterService.updateItemSizeMasters([{
  id: itemSize[0].id,
  title: "US Size 7 (17.3mm)",
  item_size_alias_name: "Standard Size 7"
}]);

// List sizes for category
const sizes = await itemSizeMasterService.listItemSizeMasters({
  grp_nos: 10
});
```

## Webhook Event

**Event Name:** `item_size_master`

**Operations:**
- `create`: Creates new item size master record
- `update`: Updates item size master and queues variant option sync
- `delete`: Removes item size master record

## Size Standards

Different regions use different size standards:

| Region | Standard | Example |
|--------|----------|---------|
| US | Numerical (1-15) | Size 7 |
| UK | Alphabetical (A-Z) | Size N |
| Europe | Circumference (mm) | Size 54 |
| India | Numerical | Size 14 |

The `item_size_alias_name` can include the size standard for clarity.
