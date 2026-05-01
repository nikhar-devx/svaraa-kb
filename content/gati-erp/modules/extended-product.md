---
title: Extended Product Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
  - catalog
---

# Extended Product Module

## Overview

The Extended Product module extends Medusa's product model with ERP-specific attributes for jewelry and precious goods. It stores detailed product information from the ERP's Style Master and links to Medusa's Product entity.

## Model: ExtendedProduct

### Database Table
`extended_product`

### Core Fields

| Field | Type | Required | Indexed | Description |
|-------|------|----------|---------|-------------|
| `id` | ID | Yes | Yes (PK) | Primary key - auto-generated |
| `style_id` | Number | Yes | Yes | Style ID from ERP |
| `style_date` | DateTime | Yes | No | Style creation date |
| `style_code` | Text | Yes | Yes | Unique style code |
| `style_alias_no` | Text | Yes | No | Alternative style number |
| `parent_style_code` | Text | No | No | Parent style reference |
| `mapping_id` | Text | Yes | Yes | Medusa Product ID mapping |

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
| `make_type` | Text | Make type (e.g., Handmade, Casting) |
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
| `dis_markup_description` | Text | Discount description |

### Cost Calculation Fields

| Field | Type | Description |
|-------|------|-------------|
| `making_cost_on` | Text | Making cost basis |
| `metal_calculation_cost_on` | Text | Metal cost calculation basis |
| `gold_loss_cost_formula` | Text | Gold loss cost formula |
| `labour_cost_formula` | Text | Labour cost formula |
| `cost_margin_per` | Float | Cost margin percentage |
| `cost_margin_is_fix` | Boolean | Is cost margin fixed |
| `actual_cost_margin_amt` | Float | Actual cost margin amount |
| `cost_margin_amt` | Float | Cost margin amount |
| `cost_discount_per` | Float | Cost discount percentage |
| `cost_discount_is_fix` | Boolean | Is cost discount fixed |
| `actual_cost_discount_amt` | Float | Actual cost discount amount |
| `cost_discount_amt` | Float | Cost discount amount |

### Price Calculation Fields

| Field | Type | Description |
|-------|------|-------------|
| `making_on` | Text | Making charge basis |
| `metal_calculation_on` | Text | Metal calculation basis |
| `gold_loss_formula` | Text | Gold loss formula |
| `labour_formula` | Text | Labour formula |
| `margin_per` | Float | Margin percentage |
| `margin_is_fix` | Boolean | Is margin fixed |
| `actual_margin_amt` | Float | Actual margin amount |
| `margin_amt` | Float | Margin amount |
| `discount_per` | Float | Discount percentage |
| `discount_is_fix` | Boolean | Is discount fixed |
| `actual_discount_amt` | Float | Actual discount amount |
| `discount_amt` | Float | Discount amount |

### Weight Fields

| Field | Type | Description |
|-------|------|-------------|
| `wax_weight` | Float | Wax weight (for casting) |
| `model_weight` | Float | Model weight |
| `gross_weight` | Float | Gross weight |
| `net_weight` | Float | Net weight |
| `gross_weights_range` | Array | Range of gross weights |
| `net_weights_range` | Array | Range of net weights |

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

### Additional Fields

| Field | Type | Description |
|-------|------|-------------|
| `stamping_instruction` | Text | Hallmark stamping instructions |
| `special_remarks` | Text | Special remarks |
| `design_production_instruction` | Text | Production instructions |
| `customer_production_instruction` | Text | Customer-specific instructions |
| `customer_remarks` | Text | Customer remarks |
| `remarks` | Text | General remarks |
| `style_history` | Text | Style history/changes |
| `web_description` | Text | Web description |
| `design_by` | Text | Designer name |
| `re_order_qty` | Number | Re-order quantity |
| `master_qty` | Number | Master quantity |
| `hsn_id` | Text | HSN ID |
| `hsn_code` | Text | HSN code |
| `metal` | Text | Metal type |
| `metal_purity` | Text | Metal purity |
| `metal_color` | Text | Metal color |
| `prices_range` | Array | Range of prices |
| `gender_data` | Array | Gender options data |
| `metal_color_variant_map` | JSON | Metal color to variant mapping |

## Relationships

- **One-to-Many** with `style_details_line` (StyleDetailsLine)
- **One-to-Many** with `style_exploration_line` (StyleExplorationLine)
- **One-to-Many** with `style_collection_line` (StyleCollectionLine)
- **One-to-One** with Medusa Product (via `mapping_id`)

## Related Models

### StyleDetailsLine
Detailed breakdown of materials used in the product:
- Raw materials (gold, diamonds, stones)
- Quantities and weights
- Cost calculations
- Setting details

### StyleExplorationLine
Product variations/explorations:
- Different stone combinations
- Alternative designs
- Variation attributes

### StyleCollectionLine
Collection associations:
- Links to Collection Master
- Collection group information

## Medusa Mapping

### Creates:
1. **Medusa Product** with basic info (title, handle, etc.)
2. **ExtendedProduct** with all ERP data
3. **Remote Link** between Product and ExtendedProduct
4. **Style detail lines**, **exploration lines**, **collection lines**

### Updates:
- Extended product fields
- Related lines (cascade delete and recreate)

### Deletes:
- ExtendedProduct (cascade deletes all related lines)
- Remote link to Product
- Note: Medusa Product is not automatically deleted

## Usage

### Service Methods

```typescript
// Get extended product service
const extendedProductService = container.resolve(EXTENDED_PRODUCT_MODULE);

// Create extended product
const extendedProduct = await extendedProductService.createExtendedProducts([{
  style_id: 10001,
  style_code: "STY001",
  style_date: new Date(),
  // ... other fields
  mapping_id: "prod_123456789"
}]);

// Update
await extendedProductService.updateExtendedProducts([{
  id: extendedProduct[0].id,
  mrp: 50000.00
}]);

// Retrieve with relations
const product = await extendedProductService.retrieveExtendedProduct(
  extendedProduct[0].id,
  {
    relations: ["style_details", "style_explorations", "style_collections"]
  }
);
```

## Workflow

The `create-extended-product-from-product` workflow handles creation:
1. Creates ExtendedProduct
2. Creates StyleDetailsLine records
3. Creates StyleExplorationLine records
4. Creates StyleCollectionLine records
5. Updates ExtendedProduct with line IDs
6. Creates remote link to Medusa Product
7. Emits sync event to OpenSearch

## Price Ranges

The plugin automatically calculates price ranges from variants:
- `prices_range`: Derived from variant MRP values
- `gross_weights_range`: Derived from variant gross weights
- `net_weights_range`: Derived from variant net weights
- `gender_data`: Derived from variant gender values
