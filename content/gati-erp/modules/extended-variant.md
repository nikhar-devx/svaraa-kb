---
title: Extended Variant Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
  - catalog
---

# Extended Variant Module

## Overview

The Extended Variant module extends Medusa's product variant model with ERP-specific attributes. It stores detailed variant information from the ERP's Party Style Master and manages inventory through Inward Master records.

## Model: ExtendedVariant

### Database Table
`extended_variant`

### Core Fields

| Field | Type | Required | Indexed | Description |
|-------|------|----------|---------|-------------|
| `id` | ID | Yes | Yes (PK) | Primary key with prefix "extended_variant" |
| `mapping_id` | Text | Yes | Yes | Medusa ProductVariant ID mapping |
| `party_style_id` | Number | Yes | Yes | Party style ID from ERP |
| `party_style_date` | DateTime | Yes | No | Party style creation date |
| `party_style_code` | Text | Yes | Yes | Party style code |
| `party_style_sku_no` | Text | Yes | Yes | Party style SKU (unique identifier) |
| `party_style_alias_no` | Text | Yes | No | Alternative style number |
| `style_id` | Number | Yes | Yes | Parent style ID |
| `style_code` | Text | Yes | Yes | Parent style code |

### Media Fields

| Field | Type | Description |
|-------|------|-------------|
| `thumbnail` | Text | Thumbnail image URL |
| `images` | Array | Array of image URLs |

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
| `manufacturer` | Text | Manufacturer name |
| `manufacturer_no` | Text | Manufacturer number |
| `make_type_no` | Number | Make type number |
| `make_type` | Text | Make type |
| `make_type_code` | Text | Make type code |
| `stock_type_no` | Number | Stock type number |
| `stock_type` | Text | Stock type |
| `stock_type_code` | Text | Stock type code |
| `item_size_id` | Number | Item size ID |
| `item_size` | Text | Item size |
| `item_size_name` | Text | Item size display name |
| `brand_no` | Number | Brand number |
| `brand` | Text | Brand |
| `brand_name` | Text | Brand display name |
| `gender_no` | Number | Gender number |
| `gender` | Enum | Gender (Men, Women, Unisex, Kids) |
| `gender_name` | Text | Gender display name |
| `shape_code` | Text | Shape code (from center stone) |
| `is_complete` | Boolean | Is style complete |
| `restricted` | Boolean | Is restricted access |
| `is_default` | Boolean | Is default variant |
| `status` | Enum | Product status (draft, proposed, published, rejected) |

### Pricing Fields

| Field | Type | Description |
|-------|------|-------------|
| `currency` | Text | Currency code |
| `rate_chart` | Text | Rate chart reference |
| `discount_markup` | Text | Discount markup code |
| `discount_markup_cost` | Text | Cost discount markup |
| `cost` | Float | Calculated cost |
| `actual_cost` | Float | Actual cost |
| `mrp` | Float | Maximum Retail Price |
| `actual_mrp` | Float | Actual MRP |
| `diamond_dis_markup_value` | Float | Diamond discount value |
| `diamond_dis_markup_type` | Text | Diamond discount type (percentage/rupees) |
| `dis_markup_description` | Text | Discount description |

### Weight Fields

| Field | Type | Description |
|-------|------|-------------|
| `wax_weight` | Float | Wax weight |
| `model_weight` | Float | Model weight |
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

### Sales Fields

| Field | Type | Description |
|-------|------|-------------|
| `sales_channels` | Array | Associated sales channel IDs |
| `outlet_ids` | Array | Available outlet/location IDs |

### Cost Breakdown Fields

| Field | Type | Description |
|-------|------|-------------|
| `metal_total_cost` | Float | Total metal cost |
| `diamond_total_cost` | Float | Total diamond cost |
| `stone_total_cost` | Float | Total stone cost |
| `cpf_total_cost` | Float | Total CPF cost |
| `metal_total_amount` | Float | Total metal amount |
| `diamond_total_amount` | Float | Total diamond amount |
| `stone_total_amount` | Float | Total stone amount |
| `cpf_total_amount` | Float | Total CPF amount |
| `actual_metal_total_amount` | Float | Actual metal total |
| `actual_diamond_total_amount` | Float | Actual diamond total |
| `actual_stone_total_amount` | Float | Actual stone total |
| `actual_cpf_total_amount` | Float | Actual CPF total |
| `actual_total_amount` | Float | Actual total amount |

### Additional Fields

| Field | Type | Description |
|-------|------|-------------|
| `stamping_instruction` | Text | Hallmark stamping instructions |
| `special_remarks` | Text | Special remarks |
| `design_production_instruction` | Text | Production instructions |
| `customer_production_instruction` | Text | Customer-specific instructions |
| `customer_remarks` | Text | Customer remarks |
| `remarks` | Text | General remarks |
| `style_history` | Text | Style history |
| `web_description` | Text | Web description |
| `design_by` | Text | Designer name |
| `master_qty` | Number | Master quantity |
| `hsn_id` | Text | HSN ID |
| `hsn_code` | Text | HSN code |
| `metal` | Text | Metal type |
| `metal_purity` | Text | Metal purity |
| `metal_color` | Text | Metal color |

## Relationships

- **One-to-Many** with `party_style_details_line` (PartyStyleDetailsLine)
- **One-to-Many** with `party_style_exploration_line` (PartyStyleExplorationLine)
- **One-to-Many** with `inward_master` (InwardMaster)
- **Many-to-One** with Medusa ProductVariant (via `mapping_id`)

## Related Models

### PartyStyleDetailsLine
Material breakdown for the variant:
- Raw materials and quantities
- Cost calculations
- Setting information

### PartyStyleExplorationLine
Variant explorations/variations

### InwardMaster
Inventory records for this variant:
- Stock receipts
- Location tracking
- Status (On Hand, Branch Issue, etc.)

## Medusa Mapping

### Creates:
1. **Medusa ProductVariant** with basic info
2. **ExtendedVariant** with all ERP data
3. **Remote Link** to ProductVariant
4. **Party style detail lines**, **exploration lines**
5. Links to **Inward Master** records

### Updates:
- Extended variant fields
- Related lines
- Status and sales channels
- Outlet IDs (based on inward status)

### Deletes:
- ExtendedVariant (cascade deletes all related lines and inward masters)
- Remote link to ProductVariant

## Outlet Management

Outlet IDs are automatically managed based on Inward Master status:
- **On Hand**: Outlet ID added to `outlet_ids`
- **Branch Issue**: Outlet ID removed from `outlet_ids`

## Usage

### Service Methods

```typescript
// Get extended variant service
const extendedVariantService = container.resolve(EXTENDED_PRODUCT_MODULE);

// Create extended variant
const extendedVariant = await extendedVariantService.createExtendedVariants([{
  party_style_id: 20001,
  party_style_code: "PS001",
  party_style_sku_no: "SKU-001-001",
  style_id: 10001,
  style_code: "STY001",
  // ... other fields
  mapping_id: "variant_123456789"
}]);

// Update
await extendedVariantService.updateExtendedVariants([{
  id: extendedVariant[0].id,
  mrp: 50000.00,
  status: "published"
}]);

// Update batch
await extendedVariantService.updateExtendedVariantBatch({
  extended_variants: [
    { id: "extvar_xxx", outlet_ids: ["cus_123", "cus_456"] }
  ]
});
```

## Inventory Integration

Extended variants integrate with Medusa's Inventory Module:
- Inward Master creates inventory levels
- Stock status derived from inward records
- Location-based inventory tracking
