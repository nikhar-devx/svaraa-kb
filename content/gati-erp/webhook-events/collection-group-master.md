---
title: Collection Group Master Webhook Events
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
  - catalog
---

# Collection Group Master Webhook Events

## Event Name
`collection_group_master`

## Overview
Collection Group Master webhook events handle organizational groupings of collections from the ERP system. Unlike other masters, this currently does not create Medusa entities but serves as reference data for collection organization.

## Event Payload Structure

```typescript
{
  operation: "create" | "update" | "delete",
  data: CollectionGroupMasterData[]
}
```

## Purpose

Collection Group Master provides:
- **Organizational Structure**: Groups collections logically
- **Reference Data**: Used by Collection Master
- **Reporting Categories**: For analytics and reports
- **Metadata Storage**: Group information for collections

## Current Behavior

**Important**: Currently, Collection Group Master does **not** create a corresponding Medusa entity. It:
- Stores data in plugin database only
- Provides reference for Collection Master
- Has `mapping_id` field reserved for future use

## Operations

### 1. CREATE Operation

**When**: New collection group added to ERP

**Flow**:
```
ERP Collection Group Created
  └── Webhook Sent (collection_group_master/create)
        └── Subscriber Receives Event
              └── Store in Collection Group Master
                    └── No Medusa Entity Created
```

**Creates**:
- **CollectionGroupMaster** record in plugin database
- No Medusa entity created
- `mapping_id` remains null (reserved)

**Data Stored**:
- `collection_group_no`: Group number
- `collection_group_code`: Group code
- `collection_group_name`: Group name
- `mapping_id`: null (reserved)

### 2. UPDATE Operation

**When**: Existing collection group modified in ERP

**Flow**:
```
ERP Collection Group Updated
  └── Webhook Sent (collection_group_master/update)
        └── Subscriber Receives Event
              └── Update Collection Group Master
                    └── No Medusa Entity to Update
```

**Updates**:
- **Group Information**: name, code, etc.
- No Medusa entity to update
- Collections referencing this group not affected

### 3. DELETE Operation

**When**: Collection group deleted from ERP

**Flow**:
```
ERP Collection Group Deleted
  └── Webhook Sent (collection_group_master/delete)
        └── Subscriber Receives Event
              └── Delete Collection Group Master
                    └── No Medusa Entity to Delete
```

**Deletes**:
- **CollectionGroupMaster** record
- No Medusa entity to delete
- Collections may reference non-existent group

## Collection Group Usage

### Reference Data
Collection Master references Collection Group:
```typescript
{
  collection_no: 5001,
  collection_name: "Festive Collection 2024",
  collection_group_no: 50,  // References Collection Group Master
  collection_group_code: "SEASONAL",
  collection_group_name: "Seasonal Collections"
}
```

### Organizational Structure
```
Collection Group: Seasonal Collections
  ├── Collection: Festive Collection 2024
  ├── Collection: Winter Collection 2024
  └── Collection: Summer Collection 2024

Collection Group: Exclusive Collections
  ├── Collection: Platinum Collection
  └── Collection: Diamond Heritage
```

## Future Enhancement

### Potential Medusa Integration
Future versions may map Collection Group to:
- **Custom Entity**: Create plugin-specific entity
- **Metadata**: Store in collection metadata
- **Tags**: Use Medusa's tag system
- **Categories**: Map to product categories

### Proposed Implementation
```typescript
// Future: Create Collection Group as Custom Entity
{
  mapping_id: "custom_entity_id",
  // ... other fields
}
```

## Data Consistency

### When Group is Deleted
Collections referencing the group:
- Keep reference to `collection_group_no`
- Group data becomes stale
- May need cleanup job

### Recommendation
Before deleting a group:
1. Check for dependent collections
2. Reassign collections to another group
3. Or mark group as inactive instead

## Common Collection Groups

| Group Code | Description |
|------------|-------------|
| SEASONAL | Season-based collections |
| FESTIVAL | Festival-specific collections |
| EXCLUSIVE | Premium/limited collections |
| BRIDAL | Wedding-related collections |
| GIFT | Gift-oriented collections |
| NEW | New arrival collections |
| SALE | Discount/promotional collections |
| THEME | Themed collections |

## Integration with Collections

### Collection Master Data
Each collection includes group information:
```typescript
{
  collection_no: 5001,
  collection_name: "Festive Collection 2024",
  // Group reference (stored for reference)
  collection_group_no: 50,
  collection_group_code: "SEASONAL",
  collection_group_name: "Seasonal Collections"
}
```

### Usage in Medusa
Group data can be used for:
- Collection filtering
- Admin organization
- Reporting categories
- Bulk operations

## Event Sequence

### Setup Flow
```
1. collection_group_master/create (optional, for reference)
     ↓
2. collection_master/create (includes group data)
     ↓
3. Products assigned to collections
```

### Update Flow
```
1. collection_group_master/update
     ↓
2. Collection Master data updated (if needed)
     ↓
3. No Medusa impact
```

## Best Practices

1. **Create Groups Early**: Define organizational structure
2. **Consistent Naming**: Use clear group names
3. **Avoid Deletion**: Mark inactive instead
4. **Document Usage**: Track which collections use which groups
5. **Plan for Future**: Structure supports future Medusa integration

## Related Events

- `collection_master`: Collections that reference groups
- Future: May have direct Medusa integration

## Reporting Use Cases

Collection Groups enable reports like:
- Sales by collection group
- Inventory by group
- Performance metrics per group
- Seasonal trend analysis

## Migration Path

If Medusa integration is added later:
1. Backfill mapping_ids for existing groups
2. Create corresponding Medusa entities
3. Update collections with group references
4. Maintain backward compatibility
