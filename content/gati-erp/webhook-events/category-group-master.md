---
title: Category Group Master Webhook Events
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
  - catalog
---

# Category Group Master Webhook Events

## Event Name
`category_group_master`

## Overview
Category Group Master webhook events handle the synchronization of top-level category groups from the ERP system to Medusa's ProductCategory entities. These are the root nodes of the product category hierarchy.

## Event Payload Structure

```typescript
{
  operation: "create" | "update" | "delete",
  data: CategoryGroupMasterData[]
}
```

## Category Hierarchy

```
Category Group Master (Level 1) ← This Event
  └── Category Master (Level 2)
        └── Sub Category Master (Level 3)
```

## Operations

### 1. CREATE Operation

**When**: New category group added to ERP

**Flow**:
```
ERP Category Group Created
  └── Webhook Sent (category_group_master/create)
        └── Subscriber Receives Event
              ├── Create Medusa ProductCategory
              │     ├── Generate Handle
              │     ├── No Parent (Root Level)
              │     └── Store Metadata
              └── Update Category Group Master mapping_id
```

**Creates**:
- **Medusa ProductCategory** (Root Level):
  - `name`: category_group_name
  - `handle`: `lab-grown-diamond-{category-group-name}`
  - `is_active`: false (disabled by default)
  - `is_internal`: false
  - `parent_category_id`: null (top level)
  - `metadata`:
    - `external_id`: grp_group_no
    - `category_group_code`: category_group_code
    - `category_group_name`: category_group_name
    - `category_group_alias_name`: category_group_alias_name
  - `additional_data`:
    - `thumbnail`: ""
    - `images`: []
    - `is_featured`: false
    - `is_custom`: false

**Impact**:
- Creates root category in Medusa
- Can have child categories
- Appears as top-level in navigation

### 2. UPDATE Operation

**When**: Existing category group modified in ERP

**Flow**:
```
ERP Category Group Updated
  └── Webhook Sent (category_group_master/update)
        └── Subscriber Receives Event
              ├── Query Category Group Master (get mapping_id)
              ├── Check mapping_id exists
              ├── Update Medusa ProductCategory
              │     ├── Update Name
              │     └── Update Metadata
              └── Log Update
```

**Updates**:
- **Category Name**: `name` field
- **Metadata**:
  - external_id
  - category_group_code
  - category_group_name
  - category_group_alias_name

**Important**:
- Only updates if `mapping_id` exists
- Parent cannot be changed (always root)
- Child categories are not affected

### 3. DELETE Operation

**When**: Category group deleted from ERP

**Flow**:
```
ERP Category Group Deleted
  └── Webhook Sent (category_group_master/delete)
        └── Subscriber Receives Event
              ├── Check mapping_id exists
              └── Delete Medusa ProductCategory
```

**Deletes**:
- **Medusa ProductCategory** (by mapping_id)

**Consequences**:
- Child categories become orphaned
- May break category hierarchy
- Products lose category associations

## Handle Generation

Handle format:
```
lab-grown-diamond-{category-group-name}
```

Example:
- Category Group: "Jewelry Items"
- Handle: `lab-grown-diamond-jewelry-items`

Transformation rules:
- Lowercase
- Replace spaces with hyphens
- Remove special characters
- Single element (no codes)

## Root Category Characteristics

### In Medusa
- `parent_category_id`: null
- Top level in category tree
- Cannot have parent category
- Can have multiple children

### Display
- Appears in main navigation
- Acts as category container
- Often used for broad classification

## Metadata Storage

### Category Group Master Metadata
```json
{
  "external_id": 10,
  "category_group_code": "JEWELRY",
  "category_group_name": "Jewelry Items",
  "category_group_alias_name": "Fine Jewelry"
}
```

### Usage
- Reverse lookups (ERP → Medusa)
- Identifies ERP source
- Stores complete ERP data

## Child Category Impact

### When Creating
```
Category Group Created
  └── Ready for Child Categories
        └── Categories Reference by grp_group_no
              └── Linked via parent_category_id
```

### When Updating
- Children remain attached
- Only name/metadata changes
- No structural impact

### When Deleting
- Children become orphaned
- Lose parent relationship
- May need reorganization

## Error Scenarios

### No Mapping ID
```
Update/Delete without mapping_id
  └── Log Warning
        └── Skip Operation
```

### Children Exist
```
Delete with Child Categories
  └── Category Deleted
        └── Children Orphaned
              └── May Need Manual Cleanup
```

## Event Sequence

### Correct Sequence
```
1. category_group_master/create (create root first)
     ↓
2. category_master/create (attach to root)
     ↓
3. sub_category_master/create (attach to category)
```

### Out of Order
If category arrives before group:
- Category created without parent
- Can be fixed by updating category later
- Or delete and recreate in correct order

## Best Practices

1. **Create Groups First**: Always start with category groups
2. **Verify Children**: Check for children before deleting
3. **Plan Hierarchy**: Design category structure before setup
4. **Activate Together**: Activate groups and children simultaneously
5. **Monitor Orphans**: Regular checks for orphaned categories

## Common Category Groups

| Group Code | Purpose |
|------------|---------|
| JEWELRY | Main jewelry categories |
| GOLDSMITH | Gold-specific items |
| DIAMOND | Diamond jewelry |
| SILVER | Silver items |
| GIFT | Gift items |
| ACCESSORIES | Jewelry accessories |

## Related Events

- `category_master`: Child categories
- `sub_category_master`: Grandchild categories
- `product-sync`: Products in these categories
