---
title: Discount Master Webhook Events
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# Discount Master Webhook Events

## Event Name
`discount_master`

## Overview
Discount Master webhook events handle the synchronization of discount and markup rules from the ERP system. These rules define pricing adjustments based on product attributes and can apply to costs or selling prices.

## Event Payload Structure

```typescript
{
  operation: "create" | "update" | "delete",
  data: DiscountMasterData[]
}
```

## Purpose

Discount Master events:
- **Store Discount Rules**: Keep pricing adjustment rules
- **Define Criteria**: Set conditions for discount application
- **Calculate Prices**: Apply discounts during price calculation
- **Manage Validity**: Handle date ranges and active status
- **Map to Promotions**: Optional Medusa PriceRule integration

## Discount Types

### By Application Target
- **Cost Discount**: Applied during cost calculation
- **Price Discount**: Applied to final selling price
- **CPF Discount**: Applied on Carat Per Fee calculations

### By Calculation Method
- **Percentage**: Fixed percentage off (e.g., 10%)
- **Fixed Amount**: Fixed discount amount (e.g., ₹500)
- **Range-Based**: Variable based on quantity/value ranges

### By Product Attributes
- **Category-Based**: Apply to specific categories
- **Make Type**: Apply to manufacturing types
- **Stock Type**: Apply to stock types
- **Material-Based**: Apply to specific raw materials
- **Style-Specific**: Apply to specific styles

## Operations

### 1. CREATE Operation

**When**: New discount rule added to ERP

**Flow**:
```
ERP Discount Master Created
  └── Webhook Sent (discount_master/create)
        └── Workflow Processes
              ├── Create Discount Master
              │     ├── Store Rule Details
              │     ├── Set Validity Dates
              │     └── Define Criteria Flags
              ├── Create Discount Details
              │     ├── Category Restrictions
              │     ├── Make Type Rules
              │     ├── Material Rules
              │     └── Range Definitions
              └── Optional: Create Medusa PriceRule
```

**Creates**:
- **DiscountMaster**: Main rule record
- **DiscountMarkupDetails**: Detailed criteria and percentages
- **Medusa PriceRule** (optional): Cart-level discount

### 2. UPDATE Operation

**When**: Existing discount rule modified in ERP

**Flow**:
```
ERP Discount Master Updated
  └── Webhook Sent (discount_master/update)
        └── Workflow Processes
              ├── Update Discount Master
              │     ├── Update Rule Details
              │     ├── Update Validity
              │     └── Update Status
              ├── Update/Recreate Details
              │     ├── Delete Old Details
              │     └── Create New Details
              └── Update Medusa PriceRule (if exists)
```

**Updates**:
- **DiscountMaster**: Rule details, dates, status
- **DiscountMarkupDetails**: Recreate with new criteria
- **PriceRule**: Update if mapped

### 3. DELETE Operation

**When**: Discount rule deleted from ERP

**Flow**:
```
ERP Discount Master Deleted
  └── Webhook Sent (discount_master/delete)
        └── Delete Discount Master
              ├── Delete All Discount Details
              └── Delete Medusa PriceRule (if exists)
```

**Deletes**:
- **DiscountMaster**: Main record
- **DiscountMarkupDetails**: All detail records
- **Medusa PriceRule**: If mapped

## Discount Structure

### DiscountMaster (Main Rule)
```typescript
{
  discount_markup_id: 1001,
  discount_markup_code: "FEST10",
  discount_markup_name: "Festival Discount 10%",
  description: "10% off on all jewelry",
  
  // Criteria Flags
  is_category: true,
  is_stock_type: false,
  is_make_type: false,
  is_raw_no: false,
  
  // Range Settings
  is_range_wise: false,
  
  // Validity
  start_date: "2024-01-01",
  end_date: "2024-12-31",
  
  // Status
  is_active: true,
  is_block_for_use: false
}
```

### DiscountMarkupDetails (Criteria)
```typescript
{
  discount_markup_id: 1001,
  
  // Category Criteria
  grp_no: 10,
  category_code: "RING",
  
  // Make Type Criteria
  make_type_no: 1,
  make_type_name: "Handmade",
  
  // Stock Type Criteria
  stock_type_no: 1,
  stock_type_name: "Ready",
  
  // Material Criteria
  raw_mit_no: 1,
  raw_mit_name: "Precious Metal",
  raw_no: 1001,
  raw_code: "GOLD-22K",
  
  // Range Criteria
  range_value_from: 10000,
  range_value_to: 50000,
  
  // Discount Value
  percentage: 10,
  is_cpf_rate: false
}
```

## Validity Rules

### Date Range
```typescript
// Active between dates
if (currentDate >= start_date && currentDate <= end_date) {
  // Discount can be applied
}
```

### Status Flags
- `is_active`: Must be true
- `is_block_for_use`: Must be false
- `default_dis_markup`: Auto-apply if true

## Application Scenarios

### Category Discount
```
Discount: 10% on Rings
Criteria:
  - is_category: true
  - grp_no: 10 (Rings category)
Application:
  - All products in Rings category get 10% off
```

### Make Type Discount
```
Discount: 5% on Handmade items
Criteria:
  - is_make_type: true
  - make_type_no: 1 (Handmade)
Application:
  - All handmade products get 5% off
```

### Material Discount
```
Discount: 15% on Platinum
Criteria:
  - is_raw_no: true
  - raw_no: 2001 (Platinum)
Application:
  - All platinum products get 15% off
```

### Range-Based Discount
```
Discount: Tiered by order value
Criteria:
  - is_range_wise: true
  - range_value_from: 50000
  - range_value_to: 100000
  - percentage: 5
Application:
  - Orders between 50K-100K get 5% off
```

## Medusa Integration

### PriceRule Mapping
Discount Master can map to Medusa PriceRule:

```typescript
{
  mapping_id: "pricerule_123456",
  // ... discount data
}
```

### Discount Application
```
Cart Checkout
  └── Check Applicable Discounts
        ├── Match Product Attributes
        ├── Check Date Validity
        ├── Calculate Discount Amount
        └── Apply to Cart Total
```

## Error Scenarios

### Overlapping Rules
```
Multiple Discounts Apply
  └── Which One to Use?
        └── Priority System:
              ├── Specific over General
              ├── Higher % over Lower %
              └── Latest Created Wins
```

### Invalid Date Range
```
Discount with Past End Date
  └── Mark as Inactive
        └── Or Auto-Expire
```

### Missing Criteria
```
Discount with No Details
  └── Cannot Apply
        └── Log Warning
```

## Event Sequence

### Discount Setup
```
1. discount_master/create
     └── Rule Created
     ↓
2. discount_markup_details/create
     └── Criteria Added
     ↓
3. Products Calculate Prices
     └── Discount Applied if Criteria Match
```

### Price Calculation Flow
```
Extended Product/Variant
  └── Calculate Cost
        ├── Base Cost
        ├── Add Markups
        ├── Apply Cost Discounts
        └── = Final Cost
  └── Calculate Price (MRP)
        ├── Base Price
        ├── Add Margins
        ├── Apply Price Discounts
        └── = Final MRP
```

## Best Practices

1. **Clear Naming**: Use descriptive discount names
2. **Date Planning**: Set realistic validity periods
3. **Specific Criteria**: Avoid overly broad rules
4. **Test Before Active**: Validate on staging
5. **Monitor Usage**: Track discount effectiveness
6. **Avoid Overlaps**: Plan discount strategy

## Related Events

- `promocode_master`: Customer-facing promo codes
- `extended-product`: Applies discounts to products
- `extended-variant`: Applies discounts to variants
- `orders`: Uses discounts in calculations

## Debugging

### Check Active Discounts
```typescript
const discounts = await discountMasterService.listDiscountMasters({
  is_active: true,
  is_block_for_use: false,
  start_date: { $lte: new Date() },
  end_date: { $gte: new Date() }
});
```

### Verify Discount Application
```typescript
// Check if discount applies to product
const applies = checkDiscountCriteria(product, discount);
console.log(`Discount ${discount.code} applies: ${applies}`);
```

### Calculate Discount Amount
```typescript
const discountAmount = calculateDiscount(product.mrp, discount);
console.log(`Discount amount: ${discountAmount}`);
```
