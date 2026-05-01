---
title: Discount Master Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# Discount Master Module

## Overview

The Discount Master module manages discount and markup rules synchronized from the ERP system. These rules define pricing adjustments based on various product attributes and can be applied to costs or selling prices.

## Model: DiscountMaster

### Database Table
`discount_master`

### Fields

| Field | Type | Required | Indexed | Description |
|-------|------|----------|---------|-------------|
| `id` | ID | Yes | Yes (PK) | Primary key with prefix |
| `discount_markup_id` | Number | Yes | Yes | Discount/markup ID from ERP |
| `discount_markup_code` | Text | Yes | Yes | Unique discount code |
| `discount_markup_name` | Text | Yes | No | Discount rule name |
| `description` | Text | Yes | No | Description of the discount |
| `default_dis_markup` | Boolean | Yes | No | Is default discount for category |
| `is_category` | Boolean | Yes | No | Applies to categories |
| `is_stock_type` | Boolean | Yes | No | Applies to stock types |
| `is_make_type` | Boolean | Yes | No | Applies to make types |
| `is_raw_no` | Boolean | Yes | No | Applies to raw materials |
| `is_raw_mit_no` | Boolean | Yes | No | Applies to material types |
| `is_on_cpf_rate` | Boolean | Yes | No | Applies on CPF rate |
| `is_on_cpf` | Boolean | Yes | No | Applies on CPF |
| `is_range_wise` | Boolean | Yes | No | Uses range-based rules |
| `range_id` | Number | No | No | Range rule ID |
| `range_name` | Text | No | No | Range rule name |
| `range_on` | Text | No | No | Range applies on field |
| `range_on_name` | Text | No | No | Range field display name |
| `start_date` | DateTime | Yes | No | Discount start date |
| `end_date` | DateTime | Yes | No | Discount end date |
| `is_block_for_use` | Boolean | Yes | No | Flag: Blocked from use |
| `is_active` | Boolean | Yes | No | Flag: Is currently active |
| `mapping_id` | Text | No | No | Medusa PriceRule ID mapping |

## Relationships

- **One-to-Many** with `discount_markup_details` (DiscountMarkupDetails)
- Maps to Medusa PriceRule (via `mapping_id`)

## Related Model: DiscountMarkupDetails

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | ID | Primary key with prefix "dmd" |
| `discount_markup_id` | Number | Parent discount ID |
| `grp_no` | Number | Category number |
| `category_code` | Text | Category code |
| `make_type_no` | Number | Make type number |
| `make_type_name` | Text | Make type name |
| `stock_type_no` | Number | Stock type number |
| `stock_type_name` | Text | Stock type name |
| `raw_mit_no` | Number | Raw material type number |
| `raw_mit_name` | Text | Raw material type name |
| `raw_no` | Number | Raw material number |
| `raw_code` | Text | Raw material code |
| `range_detail_id` | Number | Range detail ID |
| `range_value_name` | Text | Range value name |
| `range_value_from` | Number | Range start value |
| `range_value_to` | Number | Range end value |
| `range_qty` | Number | Range quantity |
| `is_cpf_rate` | Boolean | Is CPF rate calculation |
| `percentage` | Number | Discount percentage |
| `cpf_amt_percentage` | Number | CPF amount percentage |

## Discount Types

### By Product Attributes
- **Category-based**: Apply to specific product categories
- **Stock Type**: Apply to stock types (Ready, Order, etc.)
- **Make Type**: Apply to make types (Handmade, Casting, etc.)
- **Material**: Apply to specific raw materials

### By Calculation Method
- **Percentage**: Fixed percentage discount
- **Amount**: Fixed amount discount
- **Range-based**: Variable based on quantity/value ranges
- **CPF-based**: Applied on CPF (Carat Per Fee) calculations

### Application Timing
- **Cost Discount**: Applied during cost calculation
- **Price Discount**: Applied to final selling price

## Medusa Integration

Discount Master can map to Medusa's:
- **PriceRule**: For cart-level discounts
- **Promotion**: For automatic discounts
- **Metadata**: Stored in product/variant metadata

## Usage

### Service Methods

```typescript
// Get discount master service
const discountMasterService = container.resolve(DISCOUNT_MASTER_MODULE);

// Create discount master with details
const discount = await discountMasterService.createDiscountMasters([{
  discount_markup_id: 1001,
  discount_markup_code: "FEST10",
  discount_markup_name: "Festival Discount 10%",
  description: "10% discount on all jewelry",
  is_category: true,
  is_stock_type: false,
  start_date: new Date("2024-01-01"),
  end_date: new Date("2024-12-31"),
  is_active: true,
  discount_markup_details: [
    {
      grp_no: 10,
      category_code: "RING",
      percentage: 10
    }
  ]
}]);

// Update
await discountMasterService.updateDiscountMasters([{
  id: discount[0].id,
  is_active: false
}]);

// List active discounts
const discounts = await discountMasterService.listDiscountMasters({
  is_active: true,
  is_block_for_use: false
});
```

## Webhook Event

**Event Name:** `discount_master`

**Operations:**
- `create`: Creates discount rule with details
- `update`: Updates discount and detail lines
- `delete`: Removes discount and all details

## Discount Application Flow

```
Discount Master Created
  └── Discount Details Created
        └── Applied to Extended Products/Variants
              └── Cost/Price Calculation Updated
                    └── Product Prices Updated in Medusa
```

## Validation Rules

Discounts are validated based on:
- Active date range (`start_date` to `end_date`)
- `is_active` flag
- `is_block_for_use` flag
- Product attributes matching detail criteria
