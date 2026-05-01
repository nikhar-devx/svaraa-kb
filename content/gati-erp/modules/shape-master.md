---
title: Shape Master Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# Shape Master Module

## Overview

The Shape Master module manages gemstone and diamond shapes synchronized from the ERP system. Shapes define the cut geometry of stones and are essential for product specifications and variant options.

## Model: ShapeMaster

### Database Table
`shape_master`

### Fields

| Field | Type | Required | Indexed | Description |
|-------|------|----------|---------|-------------|
| `id` | ID | Yes | Yes (PK) | Primary key - auto-generated |
| `title` | Text | No | No | Display title for variant options |
| `shape_no` | Number | Yes | No | Shape number from ERP |
| `shape_for` | Text | Yes | No | Shape applies to (diamond, gemstone, etc.) |
| `shape_code` | Text | Yes | Yes | Unique shape code |
| `shape_name` | Text | Yes | No | Shape name |
| `shape_alias_name` | Text | Yes | No | Alternative/display name |

## Shape Types

### Diamond Shapes (Fancy & Round)
- Round Brilliant
- Princess
- Cushion
- Oval
- Emerald
- Asscher
- Radiant
- Pear
- Heart
- Marquise

### Gemstone Shapes
- Cabochon (smooth dome)
- Rose Cut
- Briolette
- Baguette
- Tapered Baguette
- Trillion
- Bullet

### Standard Shapes

| Shape Code | Shape Name |
|------------|------------|
| RND | Round |
| PRN | Princess |
| CUS | Cushion |
| OVL | Oval |
| EML | Emerald |
| ASR | Asscher |
| RAD | Radiant |
| PER | Pear |
| HRT | Heart |
| MRQ | Marquise |

## Relationships

- **Referenced by** ExtendedProduct (via `style_details_line.shape_no`)
- **Referenced by** ExtendedVariant (via `party_style_details_line.shape_no`)
- **Referenced by** InwardMaster (via `inward_details_line.shape_no`)
- Used in ExtendedProduct `shape_code` (derived from center stone)

## Medusa Integration

Shape Master data is used to:
1. Create/update variant option values in Medusa
2. Define product shape options
3. Determine center stone shape for products
4. Synchronize shape information across products

### Center Stone Shape

The `shape_code` in ExtendedProduct is determined from:
- The center stone's shape (where `is_center_stone` = true)
- Maps to the Shape Master `shape_code`

### Variant Option Sync

When a Shape Master title is updated:
1. Entry added to `VariantOptionSyncQueue`
2. Background job processes the queue
3. Updates all affected variant options in Medusa

## Usage

### Service Methods

```typescript
// Get shape master service
const shapeMasterService = container.resolve(SHAPE_MASTER_MODULE);

// Create shape master
const shape = await shapeMasterService.createShapeMasters([{
  shape_no: 4001,
  shape_for: "diamond",
  shape_code: "RND",
  shape_name: "Round Brilliant",
  shape_alias_name: "Classic Round Cut"
}]);

// Update (triggers variant option sync)
await shapeMasterService.updateShapeMasters([{
  id: shape[0].id,
  title: "Round Brilliant Cut",
  shape_alias_name: "Ideal Round Cut"
}]);

// List shape options
const shapes = await shapeMasterService.listShapeMasters({
  shape_for: "diamond"
});
```

## Webhook Event

**Event Name:** `shape_master`

**Operations:**
- `create`: Creates new shape master record
- `update`: Updates shape master and queues variant option sync
- `delete`: Removes shape master record

## Shape Applications

Shapes are used in various contexts:

| Application | Description |
|-------------|-------------|
| Center Stone | Determines product's primary stone shape |
| Side Stones | Accent stone shapes |
| Pavé Settings | Small stone shapes |
| All Variants | Shape options for variants |

## Impact on Products

Shape affects:
- Product imagery and 3D models
- Setting type selection
- Pricing calculations
- Inventory management
