---
title: Inward Master Webhook Events
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
  - inventory
---

# Inward Master Webhook Events

## Event Name
`inward_master`

## Overview
Inward Master webhook events handle inventory receipts and stock movements from the ERP system. These events track when jewelry items are received into inventory, update stock levels, and manage location-based availability.

## Event Payload Structure

```typescript
{
  operation: "create" | "update" | "delete",
  data: InwardMasterData[]
}
```

## Purpose

Inward Master events:
- **Track Inventory**: Record stock receipts
- **Update Stock Levels**: Modify inventory quantities
- **Manage Locations**: Track item locations
- **Update Variant Availability**: Enable/disable product variants
- **Sync to OpenSearch**: Update search index with stock status

## Operations

### 1. CREATE Operation

**When**: New stock receipt recorded in ERP

**Flow**:
```
ERP Inward Master Created
  └── Webhook Sent (inward_master/create)
        └── Workflow Processes
              ├── Fetch Inward Data
              ├── Find Extended Variant (by SKU)
              ├── Update Variant Manage Inventory
              ├── Create Inventory Items
              ├── Update Inventory Levels
              ├── Create Inward Master Record
              ├── Create Inward Detail Lines
              ├── Create Inward Exploration Lines
              ├── Create Inward Extra Charges Lines
              ├── Update Extended Variant
              │     ├── Add Inward to variant
              │     └── Update Outlet IDs
              └── Emit OpenSearch Sync Event
```

**Creates/Updates**:
- **InwardMaster**: Complete inward record
- **InwardDetailsLine**: Material details
- **InwardExplorationLine**: Variation details
- **InwardExtraChargesLine**: Additional charges
- **InventoryItem**: Medusa inventory item
- **InventoryLevel**: Stock quantities
- **ExtendedVariant**: Links inward, updates outlets

### 2. UPDATE Operation

**When**: Existing inward record modified (status change, etc.)

**Flow**:
```
ERP Inward Master Updated
  └── Webhook Sent (inward_master/update)
        └── Workflow Processes
              ├── Fetch Existing Inward
              ├── Update Inward Master Record
              ├── Update Inventory Levels (if changed)
              ├── Update Extended Variant Outlets
              │     ├── On Hand → Add Outlet
              │     └── Branch Issue → Remove Outlet
              └── Emit OpenSearch Sync Event
```

**Updates**:
- **InwardMaster**: Status, location, etc.
- **InventoryLevel**: Quantities (if applicable)
- **ExtendedVariant.outlet_ids**: Based on status

**Status Impact**:
| Status | Action |
|--------|--------|
| On Hand | Add location to outlet_ids |
| Branch Issue | Remove location from outlet_ids |
| Sold | Remove from available |
| Reserved | Keep but mark reserved |

### 3. DELETE Operation

**When**: Inward record deleted from ERP

**Flow**:
```
ERP Inward Master Deleted
  └── Webhook Sent (inward_master/delete)
        └── Workflow Processes
              ├── Fetch Inward Master
              ├── Delete Inward Detail Lines
              ├── Delete Inward Exploration Lines
              ├── Delete Inward Extra Charges Lines
              ├── Delete Inward Master
              ├── Update Inventory Levels
              ├── Update Extended Variant
              │     ├── Remove Inward Reference
              │     └── Update Outlet IDs
              └── Emit OpenSearch Sync Event
```

**Deletes**:
- **InwardMaster**: Record and all lines
- **InventoryLevel**: Adjust quantities
- **ExtendedVariant.inward_master**: Remove reference
- **ExtendedVariant.outlet_ids**: Update locations

## Stock Status Values

| Status | Description | Impact |
|--------|-------------|--------|
| On Hand | Available in stock | Item visible, purchasable |
| Branch Issue | Transferred out | Item not available at location |
| Sold | Item sold | Removed from inventory |
| Reserved | Held for customer | Not available for others |
| In Transit | Moving between locations | Temporarily unavailable |

## Inventory Integration

### Medusa Inventory Module
Inward Master integrates with:
- **InventoryItem**: Physical item record
- **InventoryLevel**: Stock at location
- **Reservation**: Hold stock for orders
- **StockLocation**: Physical locations

### Stock Location Mapping
```
Inward Master Location (ERP)
  └── Party Master (Branch)
        └── mapping_id (Medusa Customer/Stock Location)
              └── Stock Location in Medusa
```

### Quantity Management
```
Inward Master Created
  └── Inventory Level Increased
        └── Available Quantity Updated
              └── Product Becomes Available
```

## Extended Variant Updates

### Outlet ID Management
Outlet IDs represent locations where the variant is available:

```typescript
// When status is "On Hand"
extendedVariant.outlet_ids.push(location_mapping_id);

// When status is "Branch Issue"
extendedVariant.outlet_ids = outlet_ids.filter(
  id => id !== location_mapping_id
);
```

### Inward Master Link
Inward masters are linked to variants:
```typescript
{
  id: "extended_variant_id",
  inward_master: ["inward_1", "inward_2", "inward_3"],
  outlet_ids: ["location_1", "location_2"]
}
```

## Inward Marking

When an order uses an inward:
```
Order Placed
  └── Items Matched to Inwards
        └── Inward Marked
              ├── is_inward_marked: true
              ├── inward_marked_at: timestamp
              └── inward_order_id: order_id
```

This prevents:
- Double-selling
- Inventory discrepancies
- Overselling

## Workflow Steps

### Create Inward Workflow
1. **Fetch Data**: Get inward data from webhook
2. **Transform Data**: Format for Medusa
3. **Find Variants**: Match by inward_sku_no
4. **Update Variants**: Set manage_inventory = true
5. **Get Inventory**: Query current inventory levels
6. **Find Locations**: Get party masters for locations
7. **Create Inwards**: Insert inward master records
8. **Create Lines**: Insert detail/exploration/extra charges
9. **Update Inwards**: Link lines to inward masters
10. **Update Stock**: Adjust inventory levels
11. **Update Variants**: Add inward references and outlet IDs
12. **Sync OpenSearch**: Emit sync event

## Error Scenarios

### No Matching Variant
```
Inward Created
  └── No Variant Found (by SKU)
        └── Inward Stored but Not Linked
              └── Manual Reconciliation Needed
```

### No Location Mapping
```
Inward with Unknown Location
  └── Location Not Found in Party Master
        └── Use Raw Location Data
              └── May Need Manual Fix
```

### Inventory Sync Failure
```
Inward Processed
  └── Inventory Update Fails
        └── Log Error
              └── Retry or Alert Admin
```

## Event Sequence

### Stock Receipt Flow
```
1. inward_master/create
     └── Inventory Received
     ↓
2. Inventory Module Updated
     └── Stock Levels Adjusted
     ↓
3. Extended Variant Updated
     └── Outlets Updated
     ↓
4. sync-to-os-variant Emitted
     └── Search Index Updated
```

### Stock Transfer Flow
```
1. inward_master/update (status: Branch Issue)
     └── Stock Transferred Out
     ↓
2. Inventory Level Reduced
     └── Available Quantity Decreased
     ↓
3. Extended Variant Updated
     └── Outlet Removed
     ↓
4. sync-to-os-variant Emitted
     └── Search Index Updated
```

## OpenSearch Sync

When inventory changes:
```typescript
emitEventStep({
  eventName: "sync-to-os-variant",
  data: {
    type: "variant",
    operation: "update",
    id: variantIds
  }
});
```

This updates:
- Stock status in search results
- Availability filters
- Location-based availability

## Related Events

- `extended-variant`: Parent variant entity
- `product-sync`: Variant status updates
- `mark-inward`: Order-to-inward matching

## Best Practices

1. **Process FIFO**: First in, first out for inventory
2. **Validate SKUs**: Ensure SKUs match variants
3. **Track Locations**: Maintain accurate location mapping
4. **Monitor Stock**: Regular inventory reconciliation
5. **Handle Errors**: Log and alert on sync failures

## Performance Considerations

- Workflow processes in batches
- Inventory updates are atomic
- OpenSearch sync is asynchronous
- Location lookups are cached

## Debugging

### Check Inward Creation
```typescript
// Query created inward
const { data: inward } = await query.graph({
  entity: "inward_master",
  fields: ["*"],
  filters: { inward_sku_no: "SKU-001" }
});
```

### Verify Inventory Levels
```typescript
// Check inventory
const { data: levels } = await query.graph({
  entity: "inventory_level",
  fields: ["*"],
  filters: { inventory_item_id: itemId }
});
```

### Monitor Variant Outlets
```typescript
// Check variant outlets
const variant = await extendedVariantService.retrieveExtendedVariant(
  variantId,
  { relations: ["inward_master"] }
);
console.log(variant.outlet_ids);
```
