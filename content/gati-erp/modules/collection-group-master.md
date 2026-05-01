---
title: Collection Group Master Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
  - catalog
---

# Collection Group Master Module

## Overview

The Collection Group Master module manages grouping of collections. It provides a way to organize collections into logical groups for better management and navigation.

## Model: CollectionGroupMaster

### Database Table
`collection_group_master`

### Fields

| Field | Type | Required | Indexed | Description |
|-------|------|----------|---------|-------------|
| `id` | ID | Yes | Yes (PK) | Primary key - auto-generated |
| `collection_group_no` | Number | Yes | No | Unique collection group number |
| `collection_group_code` | Text | Yes | No | Collection group code |
| `collection_group_name` | Text | Yes | No | Collection group display name |
| `mapping_id` | Text | No | No | Medusa entity ID mapping (if applicable) |

## Relationships

- **One-to-Many** with Collection Master (via `collection_group_no`)

## Medusa Mapping

The Collection Group Master does **not** directly map to a Medusa entity. It serves as:
- An organizational structure within the ERP
- A reference for grouping collections
- Metadata storage for collection categorization

### Current Behavior:
- Data is stored in the plugin's database
- No automatic Medusa entity creation
- `mapping_id` is reserved for future use

## Usage

### Service Methods

```typescript
// Get collection group master service
const collectionGroupMasterService = container.resolve(COLLECTION_GROUP_MASTER_MODULE);

// Create collection group master
const collectionGroup = await collectionGroupMasterService.createCollectionGroupMasters([{
  collection_group_no: 50,
  collection_group_code: "SEASONAL",
  collection_group_name: "Seasonal Collections"
}]);

// Update
await collectionGroupMasterService.updateCollectionGroupMasters([{
  id: collectionGroup[0].id,
  collection_group_name: "Seasonal & Festival Collections"
}]);

// List collection groups
const groups = await collectionGroupMasterService.listCollectionGroupMasters();

// Delete
await collectionGroupMasterService.deleteCollectionGroupMasters([collectionGroup[0].id]);
```

## Webhook Event

**Event Name:** `collection_group_master`

**Operations:**
- `create`: Stores group data in plugin database
- `update`: Updates group information
- `delete`: Removes group record

Note: Currently no Medusa entity is created for Collection Group Master.

## Collection Hierarchy

```
Collection Group Master (Organizational Level)
  └── Collection Master (Medusa ProductCollection)
        └── Products (via ExtendedProduct.style_collections)
```

Example:
```
Seasonal Collections (Group)
  ├── Festive Collection 2024
  ├── Winter Collection 2024
  └── Summer Collection 2024

Exclusive Collections (Group)
  ├── Platinum Collection
  └── Diamond Heritage
```
