---
title: Sub Category Master Webhook Events
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
  - catalog
---

# Sub Category Master Webhook Events

## Event Name
`sub_category_master`

## Overview
Sub Category Master webhook events handle the synchronization of sub-categories (third level) from the ERP system to Medusa's ProductCategory entities. Sub-categories are the most specific classification level before individual products.

## Event Payload Structure

```typescript
{
  operation: "create" | "update" | "delete",
  data: SubCategoryMasterData[]
}
```

## Category Hierarchy

```
Category Group Master (Level 1)
  └── Category Master (Level 2)
        └── Sub Category Master (Level 3) ← This Event
              └── Products (Level 4)
```

## Operations

### 1. CREATE Operation

**When**: New sub-category added to ERP

**Flow**:
```
ERP Sub Category Created
  └── Webhook Sent (sub_category_master/create)
        └── Subscriber Receives Event
              ├── Query Category Master
              │     └── Get mapping_id (parent)
              ├── Create Medusa ProductCategory
              │     ├── Generate Handle
              │     ├── Set Parent Category
              │     └── Store Metadata
              └── Update Sub Category Master mapping_id
```

**Creates**:
- **Medusa ProductCategory** (Level 3):
  - `name`: sub_category_name
  - `handle`: `lab-grown-diamond-{sub-category-name}-{sub-category-code}-{category-code}`
  - `is_active`: false (disabled by default)
  - `is_internal`: false
  - `parent_category_id`: From Category Master's mapping_id
  - `metadata`:
    - `external_id`: sub_item_no
  - `additional_data`:
    - `thumbnail`: ""
    - `images`: []
    - `is_featured`: false
    - `is_custom`: false

**Prerequisites**:
- Category Master must exist with mapping_id
- If parent not found, sub-category is created without parent

### 2. UPDATE Operation

**When**: Existing sub-category modified in ERP

**Flow**:
```
ERP Sub Category Updated
  └── Webhook Sent (sub_category_master/update)
        └── Subscriber Receives Event
              ├── Query Sub Category Master (get mapping_id)
              ├── Check mapping_id exists
              ├── Query Category Master (current)
              ├── Update Medusa ProductCategory
              │     ├── Update Name
              │     ├── Update Parent (if changed)
              │     └── Update Metadata
              └── Log Update
```

**Updates**:
- **Sub Category Name**: `name` field
- **Parent Category**: If `grp_no` (category) changed
- **Metadata**: external_id

**Important**:
- Only updates if `mapping_id` exists
- Parent category can be changed
- Products are not affected

### 3. DELETE Operation

**When**: Sub-category deleted from ERP

**Flow**:
```
ERP Sub Category Deleted
  └── Webhook Sent (sub_category_master/delete)
        └── Subscriber Receives Event
              ├── Check mapping_id exists
              └── Delete Medusa ProductCategory
```

**Deletes**:
- **Medusa ProductCategory** (by mapping_id)

**Consequences**:
- Products lose this sub-category association
- May move to parent category only
- Navigation structure changes

## Handle Generation

Handle format:
```
lab-grown-diamond-{sub-category-name}-{sub-category-code}-{category-code}
```

Example:
- Sub Category: "Gold Rings"
- Sub Category Code: "GOLD-RING"
- Category Code: "RING"
- Handle: `lab-grown-diamond-gold-rings-gold-ring-ring`

Transformation rules:
- Lowercase
- Replace spaces with hyphens
- Remove special characters
- Include parent category code

## Parent Category Resolution

### Create Flow
1. Query `category_master` by `category_code` or `grp_no`
2. Get `mapping_id` from category
3. Use as `parent_category_id`

### Update Flow
1. Query current `sub_category_master` to get `grp_no`
2. Query `category_master` by `grp_no`
3. Get current `mapping_id`
4. Use as new `parent_category_id`

## Complete Hierarchy Example

```
Jewelry Items (Category Group)
  └── Rings (Category)
        ├── Gold Rings (Sub Category)
        │     ├── 22K Gold Band Ring (Product)
        │     ├── 22K Gold Signet Ring (Product)
        │     └── 18K Gold Wedding Ring (Product)
        ├── Diamond Rings (Sub Category)
        │     ├── Solitaire Diamond Ring (Product)
        │     ├── Halo Diamond Ring (Product)
        │     └── Three-Stone Diamond Ring (Product)
        └── Platinum Rings (Sub Category)
              ├── Platinum Wedding Band (Product)
              └── Platinum Engagement Ring (Product)
```

## Metadata Storage

### Sub Category Master Metadata
```json
{
  "external_id": 1001001
}
```

### Usage
- Reverse lookups (ERP → Medusa)
- Identifies ERP source
- Stores sub_item_no for reference

## Error Scenarios

### Missing Category
```
Create without Category
  └── Sub Category Created without Parent
        └── Orphaned (can be fixed later)
```

### No Mapping ID
```
Update/Delete without mapping_id
  └── Log Warning
        └── Skip Operation
```

### Category Changed
```
Sub Category Moved to Different Category
  └── Parent Updated
        └── Hierarchy Restructured
```

## Event Sequence

### Correct Sequence
```
1. category_group_master/create
     ↓
2. category_master/create
     ↓
3. sub_category_master/create
     ↓
4. extended-product/create (assign to sub-category)
```

### Out of Order
If sub-category arrives before category:
- Created without parent
- Update when category arrives
- Or manual assignment needed

## Best Practices

1. **Follow Hierarchy**: Always create parent before child
2. **Specific Names**: Use descriptive sub-category names
3. **Consistent Codes**: Maintain code conventions
4. **Check References**: Ensure category exists
5. **Activate Gradually**: Test hierarchy before activation

## Common Sub Categories

### Under "Rings" Category
- Gold Rings
- Diamond Rings
- Platinum Rings
- Silver Rings
- Gemstone Rings

### Under "Necklaces" Category
- Gold Chains
- Diamond Pendants
- Pearl Necklaces
- Gold Necklace Sets

### Under "Earrings" Category
- Gold Earrings
- Diamond Studs
- Drop Earrings
- Jhumkas

## Related Events

- `category_group_master`: Root level
- `category_master`: Parent categories
- `extended-product`: Products in sub-categories

## Navigation Impact

Sub-categories typically appear as:
- Left sidebar filters
- Mega menu third level
- Breadcrumb navigation
- Category page sub-filters

## URL Structure

With sub-categories, URLs become:
```
/categories/jewelry-items/rings/gold-rings
```

This provides:
- SEO-friendly structure
- Clear navigation path
- Organized product grouping
