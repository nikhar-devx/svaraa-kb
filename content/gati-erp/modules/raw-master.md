---
title: Raw Master Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# Raw Master Module

## Overview

The Raw Master module manages raw material information synchronized from the ERP system. This includes precious metals, diamonds, gemstones, and other materials used in jewelry manufacturing.

## Model: RawMaster

### Database Table
`raw_master`

### Fields

| Field | Type | Required | Indexed | Description |
|-------|------|----------|---------|-------------|
| `id` | ID | Yes | Yes (PK) | Primary key - auto-generated |
| `title` | Text | No | No | Display title for variant options |
| `raw_no` | Number | Yes | No | Raw material number from ERP |
| `raw_code` | Text | Yes | Yes | Unique raw material code |
| `raw_name` | Text | Yes | No | Raw material name |
| `raw_alias_name` | Text | Yes | No | Alternative/display name |
| `raw_mit_no` | Number | Yes | No | Raw material item type number |
| `raw_mit_name` | Text | Yes | No | Raw material item type name |
| `material_group_id` | Number | No | No | Material group ID reference |
| `material_group` | Text | No | No | Material group name |
| `material_group_name` | Text | No | No | Material group display name |

## Material Types

Common raw material types include:

### Metals
- Gold (various purities: 24K, 22K, 18K, 14K)
- Silver
- Platinum
- Palladium

### Diamonds
- Natural Diamonds
- Lab-Grown Diamonds
- Diamond melee (small diamonds)

### Gemstones
- Ruby
- Sapphire
- Emerald
- Pearl
- Other precious/semi-precious stones

### Other Materials
- Cubic Zirconia (CZ)
- Synthetic stones
- Alloys
- Solder

## Relationships

- **Referenced by** ExtendedProduct (via `style_details_line.raw_no`)
- **Referenced by** ExtendedVariant (via `party_style_details_line.raw_no`)
- **Referenced by** InwardMaster (via `inward_details_line.raw_no`)

## Medusa Integration

Raw Master data is used to:
1. Create/update variant option values in Medusa
2. Populate product option titles
3. Synchronize material information across products

### Variant Option Sync

When a Raw Master title is updated:
1. Entry added to `VariantOptionSyncQueue`
2. Background job processes the queue
3. Updates all affected variant options in Medusa
4. Updates `title` field of variant options

## Usage

### Service Methods

```typescript
// Get raw master service
const rawMasterService = container.resolve(RAW_MASTER_MODULE);

// Create raw master
const raw = await rawMasterService.createRawMasters([{
  raw_no: 1001,
  raw_code: "GOLD-22K",
  raw_name: "22 Karat Gold",
  raw_alias_name: "22K Gold",
  raw_mit_no: 1,
  raw_mit_name: "Precious Metal",
  material_group_id: 10,
  material_group: "METAL",
  material_group_name: "Metals"
}]);

// Update (triggers variant option sync)
await rawMasterService.updateRawMasters([{
  id: raw[0].id,
  title: "22K Yellow Gold",
  raw_alias_name: "Pure 22K Gold"
}]);

// List raw materials
const rawMaterials = await rawMasterService.listRawMasters({
  material_group: "METAL"
});
```

## Webhook Event

**Event Name:** `raw_master`

**Operations:**
- `create`: Creates new raw master record
- `update`: Updates raw master and queues variant option sync
- `delete`: Removes raw master record

## Material Groups

Materials are organized into groups for easier management:

| Group Code | Description |
|------------|-------------|
| METAL | Precious metals (Gold, Silver, Platinum) |
| DIAMOND | All diamond types |
| GEMSTONE | Colored gemstones |
| SYNTHETIC | Lab-created stones |
| OTHER | Other materials |

## Sync Queue Integration

When Raw Master title changes:
```
Raw Master Updated
  └── VariantOptionSyncQueue Entry Created
        └── Background Job Processes Queue
              └── Updates Variant Options in Medusa
                    └── Updates Product Data in OpenSearch
```
