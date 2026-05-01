---
title: Category Master Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
  - catalog
---

# Category Master Module

## Overview

The Category Master module manages product categories synchronized from the ERP system. Categories are organized hierarchically under Category Groups and map to Medusa's ProductCategory entities.

## Model: CategoryMaster

### Database Table
`category_master`

### Fields

| Field | Type | Required | Indexed | Description |
|-------|------|----------|---------|-------------|
| `id` | ID | Yes | Yes (PK) | Primary key - auto-generated |
| `grp_no` | Number | Yes | Yes | Category group number (parent) |
| `category_code` | Text | Yes | No | Unique category code |
| `category_name` | Text | Yes | No | Category display name |
| `category_alias_name` | Text | Yes | No | Alternative/display name |
| `grp_group_no` | Number | Yes | Yes | Category group number reference |
| `category_group_code` | Text | Yes | No | Category group code |
| `category_group_name` | Text | Yes | No | Category group name |
| `category_group_alias_name` | Text | Yes | No | Category group alias |
| `mapping_id` | Text | No | No | Medusa ProductCategory ID mapping |

## Relationships

- **Many-to-One** with Category Group Master (via `grp_group_no`)
- **Many-to-One** with Medusa ProductCategory (via `mapping_id`)
- **One-to-Many** with Sub Category Master (via `grp_no`)

## Medusa Mapping

When a Category Master record is created/updated:

### Creates:
**Medusa ProductCategory** with:
- `name`: `category_name`
- `handle`: Auto-generated kebab-case handle (format: `lab-grown-diamond-{category-name}-{category-code}-{category-group-code}`)
- `is_active`: `false` (disabled by default)
- `is_internal`: `false`
- `parent_category_id`: Mapped from Category Group Master's `mapping_id`
- `metadata`: 
  - `external_id`: `grp_no`
  - `category_code`: `category_code`
  - `category_name`: `category_name`
- `additional_data`:
  - `thumbnail`: `""`
  - `images`: `[]`
  - `is_featured`: `false`
  - `is_custom`: `false`

### Updates:
- Category name
- Parent category (if category group changes)
- Metadata fields

### Deletes:
- Medusa ProductCategory (only if `mapping_id` exists)

## Usage

### Service Methods

```typescript
// Get category master service
const categoryMasterService = container.resolve(CATEGORY_MASTER_MODULE);

// Create category master
const category = await categoryMasterService.createCategoryMasters([{
  grp_no: 1001,
  category_code: "RING",
  category_name: "Rings",
  category_alias_name: "Rings Collection",
  grp_group_no: 10,
  category_group_code: "JEWELRY",
  category_group_name: "Jewelry Items",
  category_group_alias_name: "Fine Jewelry"
}]);

// Update category master with mapping
await categoryMasterService.updateCategoryMasters([{
  id: category[0].id,
  category_name: "Gold Rings",
  mapping_id: "pcat_123456789"
}]);

// List categories
const categories = await categoryMasterService.listCategoryMasters({
  grp_group_no: 10
});
```

## Webhook Event

**Event Name:** `category_master`

**Operations:**
- `create`: Creates new Medusa product category under parent group
- `update`: Updates category name and parent relationship
- `delete`: Removes category from Medusa

See [Webhook Events - Category Master](../webhook-events/category-master.md) for detailed flow.

## Category Hierarchy

```
Category Group Master (Level 1)
  └── Category Master (Level 2)
        └── Sub Category Master (Level 3)
```

Example:
```
Jewelry Items (Category Group)
  └── Rings (Category)
        └── Gold Rings (Sub Category)
        └── Diamond Rings (Sub Category)
  └── Necklaces (Category)
        └── Gold Necklaces (Sub Category)
```
