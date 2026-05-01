---
title: Quality Master Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# Quality Master Module

## Overview

The Quality Master module manages diamond and gemstone quality grades synchronized from the ERP system. This includes clarity grades, color codes, and other quality attributes used in jewelry valuation.

## Model: QualityMaster

### Database Table
`quality_master`

### Fields

| Field | Type | Required | Indexed | Description |
|-------|------|----------|---------|-------------|
| `id` | ID | Yes | Yes (PK) | Primary key - auto-generated |
| `title` | Text | No | No | Display title for variant options |
| `qly_no` | Number | Yes | No | Quality number from ERP |
| `qly_for` | Text | Yes | No | Quality applies to (diamond, gemstone, etc.) |
| `qly_code` | Text | Yes | Yes | Unique quality code |
| `qly_name` | Text | Yes | No | Quality name |
| `qly_alias_name` | Text | Yes | No | Alternative/display name |
| `qly_clarity` | Float | No | No | Clarity grade value |
| `qly_mfg_clarity` | Float | No | No | Manufacturing clarity value |
| `is_block_for_use` | Boolean | Yes | No | Flag: Blocked from use |

## Quality Types

### Diamond Clarity Grades
- FL (Flawless)
- IF (Internally Flawless)
- VVS1, VVS2 (Very Very Slightly Included)
- VS1, VS2 (Very Slightly Included)
- SI1, SI2 (Slightly Included)
- I1, I2, I3 (Included)

### Diamond Color Grades
- D (Colorless)
- E-F (Colorless)
- G-H (Near Colorless)
- I-J (Near Colorless)
- K-M (Faint Yellow)
- N-R (Very Light Yellow)
- S-Z (Light Yellow)

### Gemstone Quality
- AAA (Highest quality)
- AA (High quality)
- A (Good quality)
- B (Commercial quality)

## Relationships

- **Referenced by** ExtendedProduct (via `style_details_line.qly_no`)
- **Referenced by** ExtendedVariant (via `party_style_details_line.qly_no`)
- **Referenced by** InwardMaster (via `inward_details_line.qly_no`)

## Medusa Integration

Quality Master data is used to:
1. Create/update variant option values in Medusa (e.g., diamond clarity options)
2. Populate product quality attributes
3. Synchronize quality information across products

### Variant Option Sync

When a Quality Master title is updated:
1. Entry added to `VariantOptionSyncQueue`
2. Background job processes the queue
3. Updates all affected variant options in Medusa
4. Updates `title` field of variant options

## Usage

### Service Methods

```typescript
// Get quality master service
const qualityMasterService = container.resolve(QUALITY_MASTER_MODULE);

// Create quality master
const quality = await qualityMasterService.createQualityMasters([{
  qly_no: 2001,
  qly_for: "diamond",
  qly_code: "VVS1",
  qly_name: "Very Very Slightly Included 1",
  qly_alias_name: "VVS1 Clarity",
  qly_clarity: 8.5,
  qly_mfg_clarity: 8.0,
  is_block_for_use: false
}]);

// Update (triggers variant option sync)
await qualityMasterService.updateQualityMasters([{
  id: quality[0].id,
  title: "VVS1 - Premium",
  is_block_for_use: true
}]);

// List quality grades
const qualities = await qualityMasterService.listQualityMasters({
  qly_for: "diamond"
});
```

## Webhook Event

**Event Name:** `quality_master`

**Operations:**
- `create`: Creates new quality master record
- `update`: Updates quality master and queues variant option sync
- `delete`: Removes quality master record

## Quality Blocking

When `is_block_for_use` is set to `true`:
- Quality grade is marked as unavailable
- May be excluded from new products
- Existing products may show warnings

## Clarity Values

The `qly_clarity` and `qly_mfg_clarity` fields store numeric values for:
- Automated quality calculations
- Pricing algorithms
- Manufacturing tolerances

Higher values typically indicate better quality.
