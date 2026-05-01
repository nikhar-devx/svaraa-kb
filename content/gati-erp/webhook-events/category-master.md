---
title: Category Master Webhook Events
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
  - catalog
---

# Category Master Webhook Events

## Event Name
`category_master`

## Overview
Category Master webhook events handle the synchronization of product categories from the ERP system to Medusa's ProductCategory entities. Categories are organized hierarchically under Category Group Masters.

## Event Payload Structure

```typescript
{
  operation: "create" | "update" | "delete",
  data: CategoryMasterData[]
}
```

## Category Hierarchy

```
Category Group Master (Level 1)
  └── Category Master (Level 2) ← This Event
        └── Sub Category Master (Level 3)
```

## Operations

### 1. CREATE Operation

**When**: New category added to ERP

**Flow**:
```
ERP Category Created
  └── Webhook Sent (category_master/create)
        └── Subscriber Receives Event
              ├── Query Category Group Master
              │     └── Get mapping_id (parent)
              ├── Create Medusa ProductCategory
              │     ├── Generate Handle
              │     ├── Set Parent Category
              │     └── Store Metadata
              └── Update Category Master mapping_id
```

**Creates**:
- **Medusa ProductCategory**:
  - `name`: category_name
  - `handle`: `lab-grown-diamond-{category-name}-{category-code}-{category-group-code}`
  - `is_active`: false (disabled by default)
  - `is_internal`: false
  - `parent_category_id`: From Category Group Master's mapping_id
  - `metadata`: 
    - `external_id`: grp_no
    - `category_code`: category_code
    - `category_name`: category_name
  - `additional_data`:
    - `thumbnail`: ""
    - `images`: []
    - `is_featured`: false
    - `is_custom`: false

**Prerequisites**:
- Category Group Master must exist with mapping_id
- If parent not found, category is created without parent

### 2. UPDATE Operation

**When**: Existing category modified in ERP

**Flow**:
```
ERP Category Updated
  └── Webhook Sent (category_master/update)
        └── Subscriber Receives Event
              ├── Query Category Master (get mapping_id)
              ├── Check mapping_id exists
              ├── Query Category Group Master (current)
              ├── Update Medusa ProductCategory
              │     ├── Update Name
              │     ├── Update Parent (if changed)
              │     └── Update Metadata
              └── Log Update
```

**Updates**:
- **Category Name**: `name` field
- **Parent Category**: If `grp_group_no` changed
- **Metadata**: 
  - external_id
  - category_code
  - category_name

**Important**: 
- Only updates if `mapping_id` exists
- Parent category can be changed
- Children (sub-categories) are not affected

### 3. DELETE Operation

**When**: Category deleted from ERP

**Flow**:
```
ERP Category Deleted
  └── Webhook Sent (category_master/delete)
        └── Subscriber Receives Event
              ├── Check mapping_id exists
              └── Delete Medusa ProductCategory
```

**Deletes**:
- **Medusa ProductCategory** (by mapping_id)

**Consequences**:
- Sub-categories may become orphaned
- Products lose this category association
- Cannot delete if products are assigned

## Handle Generation

Handle format:
```
lab-grown-diamond-{category-name}-{category-code}-{category-group-code}
```

Example:
- Category: "Rings"
- Code: "RING"
- Group Code: "JEWELRY"
- Handle: `lab-grown-diamond-rings-ring-jewelry`

Transformation rules:
- Lowercase
- Replace spaces with hyphens
- Remove special characters
- Combine all elements with hyphens

## Parent Category Resolution

### Create Flow
1. Query `category_group_master` by `grp_group_no`
2. Get `mapping_id` from category group
3. Use as `parent_category_id`

### Update Flow
1. Query current `category_master` to get `grp_group_no`
2. Query `category_group_master` by that number
3. Get current `mapping_id`
4. Use as new `parent_category_id`

## Error Scenarios

### Missing Category Group
```
Create without Category Group
  └── Category Created without Parent
        └── Orphaned Category (can be fixed later)
```

### No Mapping ID
```
Update/Delete without mapping_id
  └── Log Warning
        └── Skip Operation
```

### Parent Changed
```
Category Group Changed
  └── New Parent Searched
        └── Category Moved in Hierarchy
```

## Event Sequence

### Correct Sequence for Full Hierarchy
```
1. category_group_master/create
     ↓
2. category_master/create (references group)
     ↓
3. sub_category_master/create (references category)
```

### Out of Order Handling
If category arrives before group:
- Category created without parent
- Can be updated later when group arrives
- Manual reconciliation may be needed

## Metadata Storage

### Category Master Metadata
```json
{
  "external_id": 1001,
  "category_code": "RING",
  "category_name": "Rings"
}
```

### Usage
- Used for reverse lookups (ERP → Medusa)
- Enables updates when ERP ID is known
- Stores original ERP data

## Active Status

By default, all created categories have:
- `is_active`: false
- `is_internal`: false

This means:
- Not visible in storefront by default
- Must be manually activated
- Prevents accidental exposure

## Related Events

- `category_group_master`: Parent category groups
- `sub_category_master`: Child categories
- `product-sync`: Products in these categories

## Category Management Best Practices

1. **Create Groups First**: Always create category groups before categories
2. **Check Mappings**: Verify mapping_ids exist before updates
3. **Handle Orphans**: Monitor for orphaned categories
4. **Activate Carefully**: Review before activating categories
