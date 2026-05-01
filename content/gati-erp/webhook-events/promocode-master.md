---
title: Promocode Master Webhook Events
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# Promocode Master Webhook Events

## Event Name
`promocode_master`

## Overview
Promocode Master webhook events handle the synchronization of promotional codes and coupons from the ERP system. Promocodes provide customer-facing discounts with various criteria and limitations.

## Event Payload Structure

```typescript
{
  operation: "create" | "update" | "delete",
  data: PromocodeMasterData[]
}
```

## Purpose

Promocode Master events:
- **Store Promocodes**: Keep promotional code definitions
- **Define Restrictions**: Set criteria for code usage
- **Manage Validity**: Handle date ranges and limits
- **Generate Codes**: Create individual code instances
- **Track Usage**: Monitor code redemptions

## Promocode Types

### By Discount Type
- **Percentage**: Fixed percentage off (e.g., 15% off)
- **Fixed Amount**: Fixed discount amount (e.g., ₹500 off)

### By Application Scope
- **Global**: Applies to all products
- **Category-Specific**: Only for certain categories
- **Make Type**: Only for specific make types
- **Stock Type**: Only for specific stock types
- **Style-Specific**: Only for specific styles

### By Application Timing
- **Gross Amount**: Before any deductions
- **Net Amount**: After cost deductions
- **Final Amount**: On final selling price
- **Margin/Discount**: On margin amount

## Operations

### 1. CREATE Operation

**When**: New promocode added to ERP

**Flow**:
```
ERP Promocode Master Created
  └── Webhook Sent (promocode_master/create)
        └── Workflow Processes
              ├── Create Promocode Master
              │     ├── Store Promocode Details
              │     ├── Set Validity Dates
              │     ├── Define Calculation Method
              │     └── Set Restrictions
              ├── Create PromocodeOn (Restrictions)
              │     ├── Category Restrictions
              │     ├── Make Type Rules
              │     ├── Stock Type Rules
              │     └── Style Rules
              └── Create PromocodeSeries (Codes)
                    ├── Generate Unique Codes
                    └── Set Block Status
```

**Creates**:
- **PromocodeMaster**: Main promocode definition
- **PromocodeOn**: Restriction criteria
- **PromocodeSeriesMaster**: Individual code instances

### 2. UPDATE Operation

**When**: Existing promocode modified in ERP

**Flow**:
```
ERP Promocode Master Updated
  └── Webhook Sent (promocode_master/update)
        └── Workflow Processes
              ├── Update Promocode Master
              │     ├── Update Details
              │     ├── Update Validity
              │     └── Update Status
              ├── Update/Recreate PromocodeOn
              │     ├── Delete Old Restrictions
              │     └── Create New Restrictions
              └── Update PromocodeSeries
                    ├── Add New Codes
                    └── Update Block Status
```

**Updates**:
- **PromocodeMaster**: Dates, values, status
- **PromocodeOn**: Restriction criteria
- **PromocodeSeries**: Code instances

### 3. DELETE Operation

**When**: Promocode deleted from ERP

**Flow**:
```
ERP Promocode Master Deleted
  └── Webhook Sent (promocode_master/delete)
        └── Delete Promocode Master
              ├── Delete All PromocodeOn
              ├── Delete All PromocodeSeries
              └── Remove from Active Promotions
```

**Deletes**:
- **PromocodeMaster**: Main definition
- **PromocodeOn**: All restrictions
- **PromocodeSeries**: All code instances

## Promocode Structure

### PromocodeMaster (Definition)
```typescript
{
  promocode_id: 2001,
  promocode_type_id: 1,
  promocode_type: "Percentage Discount",
  promocode_description: "New Year Sale 2024",
  
  // Validity
  from_date: "2024-01-01",
  to_date: "2024-01-31",
  entry_date: "2023-12-15",
  
  // Discount Value
  per_amt_on: true,  // true = percentage
  per_amt_value: 15,  // 15%
  min_transaction_value: 5000,
  max_per_amt_value: 2000,
  
  // Application Scope
  is_all_branch: true,
  on_gross_amount: true,
  
  // Restrictions
  is_category: true,
  is_make_type: false,
  is_stock_type: false,
  is_style: false
}
```

### PromocodeOn (Restrictions)
```typescript
{
  promocode_id: 2001,
  
  // Category Restriction
  grp_no: 10,
  category: "Rings",
  category_code: "RING",
  
  // Make Type Restriction
  make_type_no: null,
  make_type: null,
  
  // Stock Type Restriction
  stock_type_no: null,
  stock_type: null,
  
  // Style Restriction
  style_id: null,
  style_code: null
}
```

### PromocodeSeriesMaster (Code Instances)
```typescript
{
  promocode_id: 2001,
  promocode_series_id: 1,
  promocode: "NY2024-001",
  is_block: false
}
```

## Validity Rules

### Date Range
```typescript
// Code valid between dates
if (currentDate >= from_date && currentDate <= to_date) {
  // Promocode can be used
}
```

### Transaction Value
```typescript
// Minimum order value
if (orderTotal >= min_transaction_value) {
  // Promocode can be applied
}
```

### Maximum Discount
```typescript
// Cap the discount amount
const discount = Math.min(
  calculatedDiscount,
  max_per_amt_value
);
```

## Code Series Generation

### Bulk Generation
```
Promocode Master: New Year Sale
  └── Generate 1000 Codes
        ├── NY2024-001
        ├── NY2024-002
        ├── ...
        └── NY2024-1000
```

### Unique Codes
```
Customer-Specific Codes
  ├── WELCOME-ABC123 (Customer A)
  ├── WELCOME-DEF456 (Customer B)
  └── WELCOME-GHI789 (Customer C)
```

## Application Flow

### Customer Usage
```
Customer Enters Code at Checkout
  └── Validate Code
        ├── Check Code Exists (PromocodeSeries)
        ├── Check Not Blocked (is_block)
        ├── Check Date Validity
        ├── Check Transaction Minimum
        ├── Check Product Restrictions (PromocodeOn)
        └── Calculate Discount
              └── Apply to Order
```

### Validation Steps
1. **Code Existence**: Does `promocode` exist in series?
2. **Block Status**: Is `is_block` = false?
3. **Date Valid**: Is current date within range?
4. **Minimum Met**: Is order >= `min_transaction_value`?
5. **Products Valid**: Do items match restrictions?
6. **Calculate**: Apply `per_amt_value` to appropriate amount

## Restriction Examples

### Category-Only
```typescript
{
  is_category: true,
  promocode_on: [{
    grp_no: 10,
    category_code: "RING"
  }]
}
// Only rings get discount
```

### Style-Specific
```typescript
{
  is_style: true,
  promocode_on: [{
    style_id: 10001,
    style_code: "STY001"
  }]
}
// Only specific style gets discount
```

### Global
```typescript
{
  is_category: false,
  is_make_type: false,
  is_stock_type: false,
  is_style: false
}
// Applies to all products
```

## Block/Unblock Codes

### Code Redemption
```
Customer Uses Code: NY2024-001
  └── Mark as Blocked
        └── is_block: true
              └── Cannot be reused
```

### Re-enabling Codes
```typescript
// Admin can unblock
await promocodeSeriesService.updatePromocodeSeriesMasters([{
  id: seriesId,
  is_block: false
}]);
```

## Event Sequence

### Promocode Setup
```
1. promocode_master/create
     └── Definition Created
     ↓
2. promocode_on/create
     └── Restrictions Added
     ↓
3. promocode_series/create
     └── Code Instances Generated
     ↓
4. Promocode Active and Ready to Use
```

### Customer Usage Flow
```
Customer Checkout
  └── Enters Promocode
        └── System Validates:
              ├── Code Exists? ✓
              ├── Not Used? ✓
              ├── Date Valid? ✓
              ├── Min Amount Met? ✓
              └── Products Match? ✓
                    └── Discount Applied
                          └── Code Marked Used
```

## Best Practices

1. **Clear Descriptions**: Explain what the promocode does
2. **Reasonable Limits**: Set realistic min/max values
3. **Time Limits**: Don't make codes valid forever
4. **Unique Codes**: Use hard-to-guess codes
5. **Track Usage**: Monitor redemption rates
6. **Plan Inventory**: Ensure stock for promoted items

## Related Events

- `discount_master`: Automatic discounts (vs. code-based)
- `orders`: Applies promocodes to orders
- `party_master`: Customer-specific codes

## Debugging

### Check Active Promocodes
```typescript
const promocodes = await promocodeMasterService.listPromocodeMasters({
  from_date: { $lte: new Date() },
  to_date: { $gte: new Date() }
});
```

### Validate Code
```typescript
const validateCode = async (code: string) => {
  const series = await promocodeSeriesService.listPromocodeSeriesMasters({
    promocode: code,
    is_block: false
  });
  return series.length > 0;
};
```

### Check Restrictions
```typescript
const restrictions = await promocodeOnService.listPromocodeOns({
  promocode_id: promocodeId
});
console.log(`Applies to ${restrictions.length} categories/styles`);
```

## Common Promocode Scenarios

### Welcome Offer
```
Code: WELCOME20
Discount: 20% off first order
Min: ₹1000
Valid: 30 days from signup
```

### Festival Sale
```
Code: DIWALI2024
Discount: 15% off
Min: ₹5000
Max: ₹3000
Valid: Oct 15 - Nov 15
Applies to: All jewelry
```

### Category Special
```
Code: RINGS10
Discount: 10% off rings only
Min: ₹2000
Valid: Always active
Applies to: Category "Rings" only
```

### VIP Customer
```
Code: VIP-UNIQUE-CODE (per customer)
Discount: 25% off
Min: No minimum
Valid: 1 year
Usage: One-time only
```
