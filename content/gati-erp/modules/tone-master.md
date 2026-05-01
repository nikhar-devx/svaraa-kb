---
title: Tone Master Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# Tone Master Module

## Overview

The Tone Master module manages color tones for diamonds and gemstones synchronized from the ERP system. Tones represent color variations and are used in product options and variant configurations.

## Model: ToneMaster

### Database Table
`tone_master`

### Fields

| Field | Type | Required | Indexed | Description |
|-------|------|----------|---------|-------------|
| `id` | ID | Yes | Yes (PK) | Primary key - auto-generated |
| `title` | Text | No | No | Display title for variant options |
| `tone_no` | Number | Yes | No | Tone number from ERP |
| `tone_for` | Text | Yes | No | Tone applies to (diamond, metal, etc.) |
| `tone_code` | Text | Yes | Yes | Unique tone code |
| `tone_name` | Text | Yes | No | Tone name |
| `tone_alias_name` | Text | Yes | No | Alternative/display name |
| `is_block_for_use` | Boolean | Yes | No | Flag: Blocked from use |

## Tone Types

### Diamond Color Tones
- D-E (Colorless)
- F-G (Near Colorless)
- H-I (Near Colorless)
- J-K (Faint Color)
- L-M (Light Color)

### Metal Color Tones
- Yellow Gold
- White Gold
- Rose Gold
- Two-Tone
- Three-Tone

### Gemstone Tones
- Deep Red
- Pinkish Red
- Bluish Green
- Yellowish Green
- Purple

## Relationships

- **Referenced by** ExtendedProduct (via `style_details_line.tone_no`)
- **Referenced by** ExtendedVariant (via `party_style_details_line.tone_no`)
- **Referenced by** InwardMaster (via `inward_details_line.tone_no`)

## Medusa Integration

Tone Master data is used to:
1. Create/update variant option values in Medusa
2. Define product color options
3. Synchronize tone information across products

### Variant Option Sync

When a Tone Master title is updated:
1. Entry added to `VariantOptionSyncQueue`
2. Background job processes the queue
3. Updates all affected variant options in Medusa
4. Updates `title` field of variant options

## Usage

### Service Methods

```typescript
// Get tone master service
const toneMasterService = container.resolve(TONE_MASTER_MODULE);

// Create tone master
const tone = await toneMasterService.createToneMasters([{
  tone_no: 3001,
  tone_for: "metal",
  tone_code: "YG",
  tone_name: "Yellow Gold",
  tone_alias_name: "Classic Yellow Gold",
  is_block_for_use: false
}]);

// Update (triggers variant option sync)
await toneMasterService.updateToneMasters([{
  id: tone[0].id,
  title: "18K Yellow Gold",
  tone_alias_name: "Premium Yellow Gold"
}]);

// List tone options
const tones = await toneMasterService.listToneMasters({
  tone_for: "metal"
});
```

## Webhook Event

**Event Name:** `tone_master`

**Operations:**
- `create`: Creates new tone master record
- `update`: Updates tone master and queues variant option sync
- `delete`: Removes tone master record

## Tone Applications

Tones can be applied to different materials:

| Tone For | Description |
|----------|-------------|
| `diamond` | Diamond color grades |
| `metal` | Metal color variations |
| `gemstone` | Gemstone color tones |
| `pearl` | Pearl color variations |

## Usage in Products

Tone codes are used in:
- Product variant options (e.g., "Metal Color: YG")
- Style details lines (material specifications)
- Inward details (inventory tracking)
