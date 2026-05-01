---
title: Collection Master Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
  - catalog
---

# Collection Master Module

## Overview

The Collection Master module manages product collections synchronized from the ERP system. Collections represent curated groups of products and map to Medusa's ProductCollection entities.

## Model: CollectionMaster

### Database Table
`collection_master`

### Fields

| Field | Type | Required | Indexed | Description |
|-------|------|----------|---------|-------------|
| `id` | ID | Yes | Yes (PK) | Primary key - auto-generated |
| `collection_no` | Number | Yes | No | Collection number from ERP |
| `collection_code` | Text | Yes | Yes | Unique collection code |
| `collection_name` | Text | Yes | No | Collection display name |
| `collection_group_no` | Number | Yes | No | Collection group number (parent) |
| `collection_group_code` | Text | Yes | No | Collection group code |
| `collection_group_name` | Text | Yes | No | Collection group name |
| `mapping_id` | Text | No | No | Medusa ProductCollection ID mapping |

## Relationships

- **Many-to-One** with Collection Group Master (via `collection_group_no`)
- **Many-to-One** with Medusa ProductCollection (via `mapping_id`)
- **Many-to-Many** with Products (via `style_collections` in Extended Product)

## Medusa Mapping

When a Collection Master record is created/updated:

### Creates:
**Medusa ProductCollection** with:
- `title`: `collection_name`
- `handle`: Auto-generated kebab-case handle from `collection_name`
- `metadata`:
  - `external_id`: Record `id`
- `additional_data`:
  - `thumbnail`: `""`
  - `images`: `[]`
  - `is_featured`: `false`
  - `is_active`: `false`
  - `is_custom`: `false`
  - `smart_collection_type`: `null`
  - `smart_collection_price_range`: `null`

### Updates:
- Collection title/name
- Metadata fields

### Deletes:
- Medusa ProductCollection (only if `mapping_id` exists)
- Products will be removed from the collection

## Usage

### Service Methods

```typescript
// Get collection master service
const collectionMasterService = container.resolve(COLLECTION_MASTER_MODULE);

// Create collection master
const collection = await collectionMasterService.createCollectionMasters([{
  collection_no: 5001,
  collection_code: "FEST2024",
  collection_name: "Festive Collection 2024",
  collection_group_no: 50,
  collection_group_code: "SEASONAL",
  collection_group_name: "Seasonal Collections"
}]);

// Update with mapping
await collectionMasterService.updateCollectionMasters([{
  id: collection[0].id,
  collection_name: "Diwali Festive Collection 2024",
  mapping_id: "pcol_123456789"
}]);

// List collections
const collections = await collectionMasterService.listCollectionMasters({
  collection_group_no: 50
});
```

## Webhook Event

**Event Name:** `collection_master`

**Operations:**
- `create`: Creates new product collection in Medusa
- `update`: Updates collection title
- `delete`: Removes collection from Medusa

See [Webhook Events - Collection Master](../webhook-events/collection-master.md) for detailed flow.

## Collections vs Categories

| Aspect | Collections | Categories |
|--------|-------------|------------|
| Purpose | Curated product groups | Product classification |
| Hierarchy | Flat (can have groups) | Hierarchical (3 levels) |
| Usage | Marketing/Promotions | Navigation/Browsing |
| Example | "Valentine's Day Special" | "Rings > Gold Rings" |

Collections are often used for:
- Seasonal promotions
- Featured product showcases
- Themed groupings
- Marketing campaigns
