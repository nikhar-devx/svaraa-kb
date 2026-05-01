---
title: Collection Master Webhook Events
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
  - catalog
---

# Collection Master Webhook Events

## Event Name
`collection_master`

## Overview
Collection Master webhook events handle the synchronization of product collections from the ERP system to Medusa's ProductCollection entities. Collections represent curated groups of products, often used for marketing and promotional purposes.

## Event Payload Structure

```typescript
{
  operation: "create" | "update" | "delete",
  data: CollectionMasterData[]
}
```

## Collection vs Categories

| Aspect | Collections | Categories |
|--------|-------------|------------|
| Purpose | Curated groups | Classification |
| Hierarchy | Flat (can have groups) | Hierarchical (3 levels) |
| Usage | Marketing | Navigation |
| Example | "Valentine's Day" | "Rings > Gold Rings" |

## Operations

### 1. CREATE Operation

**When**: New collection added to ERP

**Flow**:
```
ERP Collection Created
  └── Webhook Sent (collection_master/create)
        └── Subscriber Receives Event
              ├── Create Medusa ProductCollection
              │     ├── Generate Handle
              │     └── Store Metadata
              └── Update Collection Master mapping_id
```

**Creates**:
- **Medusa ProductCollection**:
  - `title`: collection_name
  - `handle`: `{collection-name}` (kebab-case)
  - `metadata`:
    - `external_id`: record id
  - `additional_data`:
    - `thumbnail`: ""
    - `images`: []
    - `is_featured`: false
    - `is_active`: false
    - `is_custom`: false
    - `smart_collection_type`: null
    - `smart_collection_price_range`: null

**Example**:
- Collection: "Festive Collection 2024"
- Handle: `festive-collection-2024`

### 2. UPDATE Operation

**When**: Existing collection modified in ERP

**Flow**:
```
ERP Collection Updated
  └── Webhook Sent (collection_master/update)
        └── Subscriber Receives Event
              ├── Query Collection Master (get mapping_id)
              ├── Check mapping_id exists
              ├── Update Medusa ProductCollection
              │     └── Update Title
              └── Log Update
```

**Updates**:
- **Collection Title**: `title` field
- **Metadata**: external_id

**Important**:
- Only updates if `mapping_id` exists
- Does not update product associations
- Handle is not changed (would break URLs)

### 3. DELETE Operation

**When**: Collection deleted from ERP

**Flow**:
```
ERP Collection Deleted
  └── Webhook Sent (collection_master/delete)
        └── Subscriber Receives Event
              ├── Check mapping_id exists
              └── Delete Medusa ProductCollection
```

**Deletes**:
- **Medusa ProductCollection** (by mapping_id)

**Consequences**:
- Products removed from collection
- Collection handle becomes available
- Any collection-specific URLs break

## Handle Generation

Handle format:
```
{collection-name}
```

Example:
- Collection: "Festive Collection 2024"
- Handle: `festive-collection-2024`

Transformation rules:
- Lowercase
- Replace spaces with hyphens
- Remove special characters

## Collection Group Reference

Collection Master includes collection group data:
- `collection_group_no`: Group identifier
- `collection_group_code`: Group code
- `collection_group_name`: Group name

However, Medusa collections are flat (no hierarchy), so this data is only stored for reference.

## Product Association

Collections are associated with products through:
- `ExtendedProduct.style_collections` (StyleCollectionLine)
- Links to Collection Master by `collection_no`

### How Products Join Collections
```
Product Created/Updated
  └── Style Collection Lines Created
        └── Reference Collection Master
              └── Product Added to Collection
```

## Metadata Storage

### Collection Master Metadata
```json
{
  "external_id": "coll_123456789"
}
```

### Additional Data
```json
{
  "thumbnail": "",
  "images": [],
  "is_featured": false,
  "is_active": false,
  "is_custom": false,
  "smart_collection_type": null,
  "smart_collection_price_range": null
}
```

## Active Status

By default:
- `is_active`: false
- `is_featured`: false

Collections must be manually activated for:
- Storefront display
- Featured collection sections
- Marketing campaigns

## Error Scenarios

### No Mapping ID
```
Update/Delete without mapping_id
  └── Log Warning
        └── Skip Operation
```

### Duplicate Handle
```
Create with Existing Handle
  └── Medusa Rejects (unique constraint)
        └── Log Error
```

## Event Sequence

### Collection Setup
```
1. collection_master/create
     ↓
2. Product Created with style_collections
     ↓
3. Products Automatically Added to Collection
```

### Collection Removal
```
1. collection_master/delete
     ↓
2. Medusa Collection Deleted
     ↓
3. Products Removed from Collection
```

## Common Collections

| Collection Name | Purpose |
|-----------------|---------|
| New Arrivals | Recently added products |
| Best Sellers | Popular products |
| Festive Special | Holiday promotions |
| Wedding Collection | Bridal jewelry |
| Gift Ideas | Gift-worthy items |
| Under ₹10,000 | Price-based collection |
| Platinum Collection | Material-specific |

## Related Events

- `collection_group_master`: Collection groups (for reference)
- `extended-product`: Products in collections
- `product-sync`: Product-collection relationships

## Marketing Use Cases

### Seasonal Campaigns
```
Diwali Collection 2024
  ├── Products tagged with collection
  ├── Featured on homepage
  └── Special pricing/discounts
```

### Promotional Events
```
Valentine's Day Special
  ├── Curated gift items
  ├── Bundle offers
  └── Email campaign targeting
```

## Best Practices

1. **Plan Collections**: Design collection strategy
2. **Consistent Naming**: Use clear, descriptive names
3. **Activate When Ready**: Don't activate empty collections
4. **Monitor Performance**: Track collection effectiveness
5. **Clean Up**: Remove outdated collections
