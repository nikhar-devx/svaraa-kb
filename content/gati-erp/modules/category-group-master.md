---
title: Category Group Master Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
  - catalog
---

# Category Group Master Module

## Overview

The Category Group Master module manages top-level category groups in the product hierarchy. These are the highest level of categorization and map to parent ProductCategory entities in Medusa.

## Model: CategoryGroupMaster

### Database Table
`category_group_master`

### Fields

| Field | Type | Required | Indexed | Description |
|-------|------|----------|---------|-------------|
| `id` | ID | Yes | Yes (PK) | Primary key - auto-generated |
| `grp_group_no` | Number | Yes | Yes | Unique category group number |
| `category_group_code` | Text | Yes | No | Category group code |
| `category_group_name` | Text | Yes | No | Category group display name |
| `category_group_alias_name` | Text | Yes | No | Alternative/display name |
| `mapping_id` | Text | No | No | Medusa ProductCategory ID mapping |

## Relationships

- **One-to-Many** with Category Master (via `grp_group_no`)
- **Many-to-One** with Medusa ProductCategory (via `mapping_id`)

## Medusa Mapping

When a Category Group Master record is created/updated:

### Creates:
**Medusa ProductCategory** (parent level) with:
- `name`: `category_group_name`
- `handle`: Auto-generated kebab-case handle (format: `lab-grown-diamond-{category-group-name}`)
- `is_active`: `false` (disabled by default)
- `is_internal`: `false`
- `parent_category_id`: `null` (top level)
- `metadata`:
  - `external_id`: `grp_group_no`
  - `category_group_code`: `category_group_code`
  - `category_group_name`: `category_group_name`
  - `category_group_alias_name`: `category_group_alias_name`
- `additional_data`:
  - `thumbnail`: `""`
  - `images`: `[]`
  - `is_featured`: `false`
  - `is_custom`: `false`

### Updates:
- Category group name
- Metadata fields

### Deletes:
- Medusa ProductCategory (only if `mapping_id` exists)
- Note: Child categories may become orphaned

## Usage

### Service Methods

```typescript
// Get category group master service
const categoryGroupMasterService = container.resolve(CATEGORY_GROUP_MASTER_MODULE);

// Create category group master
const categoryGroup = await categoryGroupMasterService.createCategoryGroupMasters([{
  grp_group_no: 10,
  category_group_code: "JEWELRY",
  category_group_name: "Jewelry Items",
  category_group_alias_name: "Fine Jewelry Collection"
}]);

// Update with mapping
await categoryGroupMasterService.updateCategoryGroupMasters([{
  id: categoryGroup[0].id,
  category_group_name: "Fine Jewelry Items",
  mapping_id: "pcat_group_123456"
}]);

// List category groups
const groups = await categoryGroupMasterService.listCategoryGroupMasters();
```

## Webhook Event

**Event Name:** `category_group_master`

**Operations:**
- `create`: Creates new parent product category in Medusa
- `update`: Updates category group name
- `delete`: Removes parent category from Medusa

See [Webhook Events - Category Group Master](../webhook-events/category-group-master.md) for detailed flow.

## Hierarchy Position

```
Category Group Master (Level 1 - Top)
  └── Category Master (Level 2)
        └── Sub Category Master (Level 3)
        └── Products (Level 4)
```

The Category Group Master serves as the root node for the entire product category tree in Medusa.
