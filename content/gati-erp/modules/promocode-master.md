---
title: Promocode Master Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# Promocode Master Module

## Overview

The Promocode Master module manages promotional codes and coupons synchronized from the ERP system. Promocodes provide customers with discounts based on various criteria and time periods.

## Model: PromocodeMaster

### Database Table
`promocode_master`

### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | ID | Yes | Primary key - auto-generated |
| `promocode_id` | Number | Yes | Promocode ID from ERP |
| `promocode_type_id` | Number | Yes | Type of promocode |
| `promocode_type` | Text | Yes | Promocode type name |
| `promocode_description` | Text | Yes | Description |
| `from_date` | DateTime | Yes | Start date |
| `to_date` | DateTime | Yes | End date |
| `entry_date` | DateTime | Yes | Entry creation date |
| `per_amt_on` | Boolean | Yes | Is percentage or amount |
| `per_amt_value` | Float | Yes | Discount value |
| `min_transaction_value` | Float | Yes | Minimum order value |
| `max_per_amt_value` | Float | Yes | Maximum discount amount |
| `is_all_branch` | Boolean | Yes | Applies to all branches |
| `assign_branch_nos` | Text | No | Specific branch numbers |
| `calculation_id` | Number | Yes | Calculation method ID |
| `calculation_name` | Text | Yes | Calculation method name |
| `on_gross_amount` | Boolean | Yes | Apply on gross amount |
| `on_net_amount` | Boolean | Yes | Apply on net amount |
| `on_last_amount` | Boolean | Yes | Apply on final amount |
| `on_margin_discount_amt` | Boolean | Yes | Apply on margin/discount |
| `on_raw_material` | Boolean | Yes | Apply on raw materials |
| `is_category` | Boolean | Yes | Restrict by category |
| `is_make_type` | Boolean | Yes | Restrict by make type |
| `is_stock_type` | Boolean | Yes | Restrict by stock type |
| `is_style` | Boolean | Yes | Restrict by style |

## Relationships

- **One-to-Many** with `promocode_on` (PromocodeOn)
- **One-to-Many** with `promocode_series` (PromocodeSeriesMaster)

## Related Models

### PromocodeOn
Defines what the promocode applies to:

| Field | Type | Description |
|-------|------|-------------|
| `id` | ID | Primary key |
| `promocode_id` | Number | Parent promocode |
| `grp_no` | Number | Category number |
| `category` | Text | Category name |
| `category_code` | Text | Category code |
| `make_type_no` | Number | Make type number |
| `make_type` | Text | Make type name |
| `make_type_code` | Text | Make type code |
| `stock_type_no` | Number | Stock type number |
| `stock_type` | Text | Stock type name |
| `stock_type_code` | Text | Stock type code |
| `style_id` | Number | Style ID |
| `style_code` | Text | Style code |

### PromocodeSeriesMaster
Individual promocode instances:

| Field | Type | Description |
|-------|------|-------------|
| `id` | ID | Primary key |
| `promocode_id` | Number | Parent promocode |
| `promocode_series_id` | Number | Series ID |
| `promocode` | Text | The actual code string |
| `is_block` | Boolean | Is code blocked/used |

## Promocode Types

### By Discount Type
- **Percentage**: Fixed percentage off (e.g., 10%)
- **Fixed Amount**: Fixed discount amount (e.g., ₹500 off)

### By Application Scope
- **Global**: Applies to all products
- **Category-specific**: Only for certain categories
- **Make Type**: Only for specific make types
- **Stock Type**: Only for specific stock types
- **Style-specific**: Only for specific styles

### By Application Timing
- **Gross Amount**: Before any discounts
- **Net Amount**: After cost deductions
- **Final Amount**: On final selling price

## Validity Rules

- **Date Range**: `from_date` to `to_date`
- **Minimum Transaction**: `min_transaction_value`
- **Maximum Discount**: `max_per_amt_value`
- **Branch Restrictions**: `is_all_branch` or `assign_branch_nos`

## Usage

### Service Methods

```typescript
// Get promocode master service
const promocodeMasterService = container.resolve(PROMOCODE_MASTER_MODULE);

// Create promocode with restrictions
const promocode = await promocodeMasterService.createPromocodeMasters([{
  promocode_id: 2001,
  promocode_type_id: 1,
  promocode_type: "Percentage Discount",
  promocode_description: "New Year Sale 2024",
  from_date: new Date("2024-01-01"),
  to_date: new Date("2024-01-31"),
  per_amt_on: true,  // percentage
  per_amt_value: 15,  // 15%
  min_transaction_value: 5000,
  max_per_amt_value: 2000,
  is_all_branch: true,
  on_gross_amount: true,
  is_category: true,
  promocode_on: [
    {
      grp_no: 10,
      category: "Rings",
      category_code: "RING"
    }
  ],
  promocode_series: [
    { promocode: "NY2024-001", is_block: false },
    { promocode: "NY2024-002", is_block: false }
  ]
}]);

// Update
await promocodeMasterService.updatePromocodeMasters([{
  id: promocode[0].id,
  to_date: new Date("2024-02-15")
}]);

// List active promocodes
const promocodes = await promocodeMasterService.listPromocodeMasters({
  from_date: { $lte: new Date() },
  to_date: { $gte: new Date() }
});
```

## Webhook Event

**Event Name:** `promocode_master`

**Operations:**
- `create`: Creates promocode with series and restrictions
- `update`: Updates promocode validity and restrictions
- `delete`: Removes promocode and all related records

## Promocode Usage Flow

```
Customer Enters Promocode
  └── Validate Code (check is_block)
        └── Check Restrictions (category, date, amount)
              └── Calculate Discount
                    └── Apply to Cart/Order
```

## Promocode Series Generation

Multiple codes can be generated for a single promotion:
- Unique codes for individual customers
- Bulk codes for marketing campaigns
- One-time use codes
- Time-limited codes

Example Series:
- `NY2024-001` through `NY2024-1000`
- `WELCOME-ABC123` (unique per customer)
- `FLASH-50OFF` (campaign code)
