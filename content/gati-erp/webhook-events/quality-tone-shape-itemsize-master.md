---
title: Master Data Webhook Events (Quality, Tone, Shape, Item Size)
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# Master Data Webhook Events (Quality, Tone, Shape, Item Size)

## Overview
This document covers webhook events for master data that follow similar patterns:
- Quality Master
- Tone Master
- Shape Master
- Item Size Master

These masters work as variant option sources and trigger sync queue updates when changed.

---

## Quality Master

### Event Name
`quality_master`

### Purpose
Manages diamond and gemstone quality grades (clarity, color grades).

### Event Payload
```typescript
{
  operation: "create" | "update" | "delete",
  data: QualityMasterData[]
}
```

### Quality Types
- **Diamond Clarity**: FL, IF, VVS1, VVS2, VS1, VS2, SI1, SI2, I1, I2, I3
- **Diamond Color**: D, E, F, G, H, I, J, K, L, M
- **Gemstone Quality**: AAA, AA, A, B

### Create Flow
```
ERP Quality Created
  └── Create Quality Master Record
        └── Used in Product Quality Specs
```

### Update Flow
```
ERP Quality Updated
  └── Update Quality Master
        └── Title Changed?
              ├── Create Queue Entry
              └── Update Variant Options (Diamond Clarity)
```

### Key Fields
- `qly_no`: Quality number
- `qly_for`: Applies to (diamond, gemstone)
- `qly_code`: Quality code (VVS1, AAA, etc.)
- `qly_name`: Quality name
- `qly_clarity`: Numeric clarity value
- `is_block_for_use`: Block flag

### Usage
- Diamond clarity options in variants
- Gemstone quality specifications
- Pricing calculations

---

## Tone Master

### Event Name
`tone_master`

### Purpose
Manages color tones for diamonds, metals, and gemstones.

### Event Payload
```typescript
{
  operation: "create" | "update" | "delete",
  data: ToneMasterData[]
}
```

### Tone Types
- **Metal Colors**: Yellow Gold, White Gold, Rose Gold, Two-Tone
- **Diamond Colors**: Color grades as tones
- **Gemstone Tones**: Deep Red, Pinkish Red, etc.

### Create Flow
```
ERP Tone Created
  └── Create Tone Master Record
        └── Used in Product Color Options
```

### Update Flow
```
ERP Tone Updated
  └── Update Tone Master
        └── Title Changed?
              ├── Create Queue Entry
              └── Update Variant Options (Metal Color)
```

### Key Fields
- `tone_no`: Tone number
- `tone_for`: Applies to (diamond, metal, gemstone)
- `tone_code`: Tone code (YG, WG, RG, etc.)
- `tone_name`: Tone name
- `is_block_for_use`: Block flag

### Usage
- Metal color options (Yellow Gold, White Gold)
- Diamond color options
- Product variation options

---

## Shape Master

### Event Name
`shape_master`

### Purpose
Manages gemstone and diamond cut shapes.

### Event Payload
```typescript
{
  operation: "create" | "update" | "delete",
  data: ShapeMasterData[]
}
```

### Shape Types
- **Round**: Round Brilliant
- **Fancy**: Princess, Cushion, Oval, Emerald, etc.
- **Step Cut**: Emerald, Asscher
- **Mixed**: Radiant

### Create Flow
```
ERP Shape Created
  └── Create Shape Master Record
        └── Used in Product Shape Specs
```

### Update Flow
```
ERP Shape Updated
  └── Update Shape Master
        └── Title Changed?
              ├── Create Queue Entry
              └── Update Variant Options (Stone Shape)
```

### Key Fields
- `shape_no`: Shape number
- `shape_for`: Applies to (diamond, gemstone)
- `shape_code`: Shape code (RND, PRN, CUS, etc.)
- `shape_name`: Shape name

### Center Stone Shape
ExtendedProduct `shape_code` is derived from center stone's shape (where `is_center_stone` = true).

### Usage
- Center stone shape identification
- Side stone shape options
- Product 3D model selection

---

## Item Size Master

### Event Name
`item_size_master`

### Purpose
Manages size options for jewelry items (rings, bracelets, necklaces).

### Event Payload
```typescript
{
  operation: "create" | "update" | "delete",
  data: ItemSizeMasterData[]
}
```

### Size Types
- **Ring Sizes**: US sizes 5-15 (including half sizes)
- **Bracelet Sizes**: Small, Medium, Large (inches)
- **Necklace Lengths**: Choker, Princess, Matinee, Opera
- **Bangle Sizes**: Inner diameter or circumference

### Create Flow
```
ERP Item Size Created
  └── Create Item Size Master Record
        └── Used in Product Size Options
```

### Update Flow
```
ERP Item Size Updated
  └── Update Item Size Master
        └── Title Changed?
              ├── Create Queue Entry
              └── Update Variant Options (Size)
```

### Key Fields
- `item_size_id`: Size ID
- `item_size_code`: Size code (RING-7, etc.)
- `item_size_name`: Size name
- `grp_nos`: Associated category groups

### Category Group Association
```typescript
{
  item_size_code: "RING-7",
  item_size_name: "Ring Size 7",
  grp_nos: [10, 11, 12]  // Rings, Bands, etc.
}
```

### Usage
- Size variant options
- Category-specific sizes
- Inventory management by size

---

## Common Patterns

### Sync Queue Integration
All four masters use the Variant Option Sync Queue:

```
Master Title Updated
  └── Create Queue Entry
        ├── master_type: [QualityMst|ToneMst|ShapeMst|ItemSizeMst]
        ├── master_code: code
        ├── old_title: previous
        ├── new_title: updated
        └── status: pending
```

### Background Processing
```
Queue Entry Picked Up
  └── Find Affected Variants
        ├── Query by master code
        ├── Update option titles
        └── Sync to OpenSearch
```

### Master Type Enum
```typescript
type MasterType = 
  | "RawMst" 
  | "QualityMst" 
  | "ToneMst" 
  | "ItemSizeMst" 
  | "ShapeMst";
```

---

## Usage in Products

### ExtendedProduct / ExtendedVariant
All masters are referenced in product specifications:

```typescript
{
  // In Style Details Line or Party Style Details Line
  raw_no: 1001,           // Raw Master
  raw_code: "GOLD-22K",
  
  qly_no: 2001,           // Quality Master
  qly_code: "VVS1",
  
  tone_no: 3001,          // Tone Master
  tone_code: "YG",
  
  shape_no: 4001,         // Shape Master
  shape_code: "RND",
  
  size_no: 5001,          // Item Size Master
  size: "7"
}
```

### Variant Options
Masters become product options:
- **Quality**: Diamond Clarity (VVS1, VS1, etc.)
- **Tone**: Metal Color (Yellow Gold, White Gold)
- **Shape**: Center Stone Shape (Round, Princess)
- **Size**: Ring/Item Size (7, 7.5, 8)

---

## Best Practices

### For All Masters
1. **Standardize Codes**: Use consistent naming conventions
2. **Update Titles Carefully**: Changes affect many products
3. **Monitor Sync Queue**: Check for processing failures
4. **Use Block Flags**: Disable rather than delete
5. **Document Usage**: Track where codes are used

### Quality Master
- Maintain clarity value consistency
- Follow GIA standards for diamonds
- Use consistent gemstone grading

### Tone Master
- Standardize metal color names
- Consider regional preferences
- Document two-tone combinations

### Shape Master
- Use industry-standard codes
- Document fancy shapes
- Maintain shape images

### Item Size Master
- Follow regional size standards
- Document size conversions
- Associate with correct categories

---

## Related Events

- `variant-option-sync-queue`: Background sync
- `extended-product`: Uses master data
- `extended-variant`: Uses master data in options
- `inward-master`: References masters in details
