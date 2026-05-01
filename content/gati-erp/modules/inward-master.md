---
title: Inward Master Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
  - inventory
---

# Inward Master Module

## Overview

The Inward Master module tracks inventory receipts and stock movements from the ERP system. It records when jewelry items are received into inventory, their location, and current status.

## Model: InwardMaster

### Database Table
`inward_master`

### Core Fields

| Field | Type | Required | Indexed | Description |
|-------|------|----------|---------|-------------|
| `id` | ID | Yes | Yes (PK) | Primary key with prefix "inward" |
| `jewel_id` | Number | Yes | Yes | Jewel/Item ID from ERP |
| `branch_no` | Text | Yes | Yes | Branch number |
| `branch_name` | Text | Yes | Yes | Branch name |
| `jewel_code` | Text | Yes | Yes | Jewel code |
| `inward_sku_no` | Text | Yes | Yes | Inward SKU number |
| `inward_date` | DateTime | Yes | No | Date of inward/receipt |
| `style_id` | Number | Yes | No | Parent style ID |
| `style_code` | Text | Yes | No | Parent style code |
| `location_id` | Text | Yes | Yes | Location identifier |
| `location` | Text | No | No | Location name |
| `status_id` | Text | Yes | No | Status ID |
| `status` | Text | Yes | Yes | Stock status (On Hand, Branch Issue, etc.) |

### Category Fields

| Field | Type | Description |
|-------|------|-------------|
| `grp_group_no` | Number | Category group number |
| `category_group_name` | Text | Category group name |
| `category_group_code` | Text | Category group code |
| `grp_no` | Number | Category number |
| `category` | Text | Category name |
| `category_code` | Text | Category code |
| `sub_itm_no` | Number | Sub-category number |
| `sub_category` | Text | Sub-category name |
| `sub_category_code` | Text | Sub-category code |

### Product Attributes

| Field | Type | Description |
|-------|------|-------------|
| `manufacturer` | Text | Manufacturer |
| `make_type_no` | Number | Make type number |
| `make_type` | Text | Make type |
| `make_type_code` | Text | Make type code |
| `stock_type_no` | Number | Stock type number |
| `stock_type` | Text | Stock type |
| `stock_type_code` | Text | Stock type code |
| `item_size_id` | Number | Item size ID |
| `item_size` | Text | Item size |
| `item_size_name` | Text | Item size name |
| `brand_no` | Number | Brand number |
| `brand` | Text | Brand |
| `brand_name` | Text | Brand name |
| `gender_no` | Number | Gender number |
| `gender` | Enum | Gender |
| `gender_name` | Text | Gender name |

### Pricing Fields

| Field | Type | Description |
|-------|------|-------------|
| `currency` | Text | Currency code |
| `rate_chart` | Text | Rate chart reference |
| `discount_markup` | Text | Discount markup code |
| `discount_markup_cost` | Text | Cost discount markup |
| `cost` | Float | Cost price |
| `actual_cost` | Float | Actual cost |
| `mrp` | Float | Maximum Retail Price |
| `actual_mrp` | Float | Actual MRP |

### Weight Fields

| Field | Type | Description |
|-------|------|-------------|
| `gross_weight` | Float | Gross weight |
| `net_weight` | Float | Net weight |

### Stone/Diamond Fields

| Field | Type | Description |
|-------|------|-------------|
| `total_diamond_pcs` | Number | Total diamond pieces |
| `total_diamond_weight` | Float | Total diamond weight |
| `total_client_diamond_pcs` | Number | Client diamond pieces |
| `total_client_diamond_weight` | Float | Client diamond weight |
| `total_stone_pcs` | Number | Total stone pieces |
| `total_stone_weight` | Float | Total stone weight |
| `total_cz_pcs` | Number | Total CZ pieces |
| `total_cz_weight` | Float | Total CZ weight |

### Tracking Fields

| Field | Type | Description |
|-------|------|-------------|
| `is_inward_marked` | Boolean | Is marked against an order |
| `inward_marked_at` | DateTime | When marked |
| `inward_order_id` | Text | Associated order ID |

### Certification Fields

| Field | Type | Description |
|-------|------|-------------|
| `hsn_id` | Number | HSN ID |
| `hsn_code` | Text | HSN code |
| `hall_mark_id` | Number | Hallmark ID |
| `lab_code` | Text | Laboratory code |
| `lab_name` | Text | Laboratory name |
| `certificate_no` | Text | Certificate number |

### Assignment Fields

| Field | Type | Description |
|-------|------|-------------|
| `stock_assign_party_no` | Text | Assigned to party |
| `order_proceed_item_id` | Number | Order proceed item ID |
| `remarks` | Text | Remarks |

## Relationships

- **Many-to-One** with ExtendedVariant (via `inward_sku_no` → `party_style_sku_no`)
- **One-to-Many** with `inward_details_line` (InwardDetailsLine)
- **One-to-Many** with `inward_exploration_line` (InwardExplorationLine)
- **One-to-Many** with `inward_extra_charges_line` (InwardExtraChargesLine)

## Related Models

### InwardDetailsLine
Material details for this inward:
- Item specifications
- Weights and quantities
- Cost calculations

### InwardExplorationLine
Variation details:
- Exploration attributes
- Different stone combinations

### InwardExtraChargesLine
Additional charges:
- Extra material costs
- Special processing fees

## Stock Status Values

| Status | Description |
|--------|-------------|
| `On Hand` | Item is in stock at location |
| `Branch Issue` | Item transferred to another branch |
| `Sold` | Item has been sold |
| `Reserved` | Item reserved for customer/order |
| `In Transit` | Item is being transferred |

## Medusa Integration

Inward Master integrates with:
1. **Inventory Module**: Creates/updates inventory levels
2. **ExtendedVariant**: Updates outlet_ids based on status
3. **Stock Location**: Maps branches to stock locations

### Inventory Flow

```
Inward Master Created
  └── Find Extended Variant (by SKU)
        ├── Create Inventory Item (if new)
        ├── Update Inventory Level
        ├── Update Variant outlet_ids
        └── Emit Sync Event to OpenSearch
```

### Outlet Management

Based on `status`:
- **On Hand**: Add location's `mapping_id` to variant's `outlet_ids`
- **Branch Issue**: Remove location's `mapping_id` from variant's `outlet_ids`

## Usage

### Service Methods

```typescript
// Get inward master service
const inwardMasterService = container.resolve(INWARD_MASTER_MODULE);

// Create inward master
const inward = await inwardMasterService.createInwardMasters([{
  jewel_id: 10001,
  branch_no: "BR001",
  branch_name: "Main Branch",
  jewel_code: "JWL-001",
  inward_sku_no: "SKU-001-001-A",
  inward_date: new Date(),
  style_id: 10001,
  style_code: "STY001",
  location_id: "LOC001",
  location: "Showroom A",
  status: "On Hand",
  mrp: 50000.00,
  gross_weight: 5.5,
  net_weight: 5.2
}]);

// Update status
await inwardMasterService.updateInwardMasters([{
  id: inward[0].id,
  status: "Branch Issue"
}]);

// List by status
const onHandItems = await inwardMasterService.listInwardMasters({
  status: "On Hand"
});
```

## Webhook Event

**Event Name:** `inward_master`

**Operations:**
- `create`: Creates inward record and updates inventory
- `update`: Updates inward status and inventory
- `delete`: Removes inward and adjusts inventory

## Inward Marking

When an order is placed:
1. System finds available inwards matching order items
2. Marks inwards as `is_inward_marked: true`
3. Records `inward_order_id` and `inward_marked_at`
4. Updates inventory levels

This prevents double-selling and ensures accurate stock tracking.
