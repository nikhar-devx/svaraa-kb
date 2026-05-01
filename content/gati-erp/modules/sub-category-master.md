---
title: Sub Category Master Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
  - catalog
---

# Sub Category Master Module

## Overview

The Sub Category Master module manages sub-categories (third level) in the product hierarchy. Sub-categories are children of Category Masters and represent the most specific classification level before individual products.

## Model: SubCategoryMaster

### Database Table
`sub_category_master`

### Fields

| Field | Type | Required | Indexed | Description |
|-------|------|----------|---------|-------------|
| `id` | ID | Yes | Yes (PK) | Primary key - auto-generated |
| `sub_item_no` | Number | Yes | Yes | Unique sub-item number |
| `sub_category_code` | Text | Yes | Yes | Unique sub-category code |
| `sub_category_name` | Text | Yes | No | Sub-category display name |
| `sub_category_alias_name` | Text | Yes | Yes | Alternative/display name |
| `grp_no` | Number | Yes | Yes | Parent category number |
| `category_code` | Text | Yes | No | Parent category code |
| `category_name` | Text | Yes | No | Parent category name |
| `category_alias_name` | Text | Yes | Yes | Parent category alias |
| `mapping_id` | Text | No | No | Medusa ProductCategory ID mapping |
| `sub_category_alias_mapping_id` | Text | No | No | Alias mapping ID (if applicable) |

## Relationships

- **Many-to-One** with Category Master (via `grp_no`)
- **Many-to-One** with Medusa ProductCategory (via `mapping_id`)

## Medusa Mapping

When a Sub Category Master record is created/updated:

### Creates:
**Medusa ProductCategory** (third level) with:
- `name`: `sub_category_name`
- `handle`: Auto-generated kebab-case handle (format: `lab-grown-diamond-{sub-category-name}-{sub-category-code}-{category-code}`)
- `is_active`: `false` (disabled by default)
- `is_internal`: `false`
- `parent_category_id`: Mapped from Category Master's `mapping_id`
- `metadata`:
  - `external_id`: `sub_item_no`

### Updates:
- Sub-category name
- Parent category relationship (if category changes)
- Metadata fields

### Deletes:
- Medusa ProductCategory (only if `mapping_id` exists)

## Usage

### Service Methods

```typescript
// Get sub category master service
const subCategoryMasterService = container.resolve(SUB_CATEGORY_MASTER_MODULE);

// Create sub category master
const subCategory = await subCategoryMasterService.createSubCategoryMasters([{
  sub_item_no: 1001001,
  sub_category_code: "GOLD-RING",
  sub_category_name: "Gold Rings",
  sub_category_alias_name: "Pure Gold Rings",
  grp_no: 1001,
  category_code: "RING",
  category_name: "Rings",
  category_alias_name: "Ring Collection"
}]);

// Update with mapping
await subCategoryMasterService.updateSubCategoryMasters([{
  id: subCategory[0].id,
  sub_category_name: "22K Gold Rings",
  mapping_id: "pcat_sub_123456"
}]);

// List sub categories
const subCategories = await subCategoryMasterService.listSubCategoryMasters({
  grp_no: 1001
});
```

## Webhook Event

**Event Name:** `sub_category_master`

**Operations:**
- `create`: Creates new sub-category under parent category
- `update`: Updates sub-category name and parent
- `delete`: Removes sub-category from Medusa

See [Webhook Events - Sub Category Master](../webhook-events/sub-category-master.md) for detailed flow.

## Complete Category Hierarchy

```
Category Group Master (Level 1)
  └── Category Master (Level 2)
        └── Sub Category Master (Level 3)
              └── Products (Level 4)
```

Example:
```
Jewelry Items
  └── Rings
        ├── Gold Rings
        │     └── Product: "22K Gold Band Ring"
        ├── Diamond Rings
        │     └── Product: "Solitaire Diamond Ring"
        └── Platinum Rings
              └── Product: "Platinum Wedding Band"
```

## Handle Generation Pattern

Sub-category handles follow the pattern:
```
lab-grown-diamond-{sub-category-name}-{sub-category-code}-{category-code}
```

Example:
- Sub Category: "Gold Rings" (GOLD-RING)
- Parent Category: "Rings" (RING)
- Handle: `lab-grown-diamond-gold-rings-gold-ring-ring`
