---
title: Shipping Info Master Webhook Events
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# Shipping Info Master Webhook Events

## Event Name
`shipping_info_master`

## Overview
Shipping Info Master webhook events handle the synchronization of additional shipping addresses from the ERP system to Medusa's CustomerAddress entities. These are secondary addresses associated with parties/customers.

## Event Payload Structure

```typescript
{
  operation: "create" | "update" | "delete",
  data: string[]  // Array of Shipping Info Master IDs
}
```

**Note**: Unlike other events, Shipping Info Master receives an array of IDs rather than full data objects. The subscriber queries the full record using these IDs.

## Relationship to Party Master

```
Party Master (Customer)
  ├── Default Address (from party_master)
  ├── Shipping Info 1 (shipping_info_master)
  ├── Shipping Info 2 (shipping_info_master)
  └── Shipping Info 3 (shipping_info_master)
```

## Operations

### 1. CREATE Operation

**When**: New shipping address added to ERP for a party

**Flow**:
```
ERP Shipping Info Created
  └── Webhook Sent (shipping_info_master/create)
        └── Subscriber Receives Event
              ├── Query Shipping Info Master (by ID)
              ├── Query Party Master (by party_no)
              │     └── Get mapping_id (customer_id)
              ├── Create Medusa CustomerAddress
              │     ├── Set Customer ID
              │     ├── Format Address
              │     └── Set State Code
              └── Update Shipping Info Master mapping_id
```

**Creates**:
- **Medusa CustomerAddress**:
  - `customer_id`: From Party Master's mapping_id
  - `address_name`: shipping_name
  - `company`: shipping_name
  - `first_name`: Party's firm_name
  - `last_name`: Party's firm_name
  - `address_1`: address1
  - `address_2`: address2
  - `city`: city
  - `province`: State code (e.g., "MH" for Maharashtra)
  - `country_code`: "in" (India)
  - `postal_code`: pin_code
  - `is_default_shipping`: false
  - `is_default_billing`: false
  - `metadata`:
    - `external_id`: Shipping Info Master ID

**Prerequisites**:
- Party Master must exist with mapping_id
- Party must have Medusa Customer created

### 2. UPDATE Operation

**When**: Existing shipping info modified in ERP

**Flow**:
```
ERP Shipping Info Updated
  └── Webhook Sent (shipping_info_master/update)
        └── Subscriber Receives Event
              ├── Query Shipping Info Master (by ID)
              ├── Check mapping_id exists
              ├── Update Medusa CustomerAddress
              │     ├── Update Address Fields
              │     └── Update State Code
              └── Log Update
```

**Updates**:
- **Address Lines**: address1, address2
- **City**: city
- **State**: province (converted to code)
- **Pin Code**: postal_code
- **Company**: shipping_name
- **Address Name**: shipping_name

**Important**:
- Only updates if `mapping_id` exists
- Customer cannot be changed
- Default flags remain unchanged

### 3. DELETE Operation

**When**: Shipping info deleted from ERP

**Flow**:
```
ERP Shipping Info Deleted
  └── Webhook Sent (shipping_info_master/delete)
        └── Subscriber Receives Event
              ├── Query Shipping Info Master (by ID)
              ├── Check mapping_id exists
              └── Delete Medusa CustomerAddress
```

**Deletes**:
- **Medusa CustomerAddress** (by mapping_id)

**Important**:
- Only deletes if `mapping_id` exists
- Does not affect customer
- Does not affect other addresses

## State Code Conversion

### Indian State Codes
The plugin converts full state names to ISO state codes:

| State Name | Code |
|------------|------|
| Maharashtra | MH |
| Gujarat | GJ |
| Rajasthan | RJ |
| Delhi | DL |
| Karnataka | KA |
| Tamil Nadu | TN |
| West Bengal | WB |
| Uttar Pradesh | UP |
| Punjab | PB |
| Haryana | HR |
| Andhra Pradesh | AP |
| Kerala | KL |
| Madhya Pradesh | MP |
| Odisha | OD |
| Bihar | BR |
| Telangana | TG |
| Assam | AS |
| Jharkhand | JH |
| Chhattisgarh | CG |
| Uttarakhand | UK |
| Goa | GA |
| Himachal Pradesh | HP |
| Tripura | TR |
| Meghalaya | ML |
| Manipur | MN |
| Nagaland | NL |
| Arunachal Pradesh | AR |
| Mizoram | MZ |
| Sikkim | SK |

### Conversion Logic
```typescript
const getStateCode = (stateName: string): string => {
  const stateMap: Record<string, string> = {
    "Maharashtra": "MH",
    "Gujarat": "GJ",
    // ... etc
  };
  return stateMap[stateName] || stateName;
};
```

## Address Structure

### Shipping Info Master Address
```
address1: 123 Industrial Area
address2: Phase 2, Sector 5
city: Mumbai
state: Maharashtra
country: India
pin_code: 400001
```

### Medusa Customer Address
```
address_1: 123 Industrial Area
address_2: Phase 2, Sector 5
city: Mumbai
province: MH
country_code: in
postal_code: 400001
```

## Metadata Storage

### Customer Address Metadata
```json
{
  "external_id": "ship_123456789"
}
```

This links back to the Shipping Info Master record.

## Error Scenarios

### Missing Party
```
Create without Party
  └── Log Error
        └── Skip (no customer to attach to)
```

### Party Not Mapped
```
Create with Unmapped Party
  └── Log Warning
        └── Skip (no mapping_id)
```

### No Mapping ID
```
Update/Delete without mapping_id
  └── Log Warning
        └── Skip Operation
```

## Multiple Addresses per Customer

A single party can have multiple shipping addresses:

```
Party: ABC Jewelers
  ├── Default Address (Main Office)
  │     └── From party_master
  ├── Warehouse Location
  │     └── From shipping_info_master
  ├── Showroom Address
  │     └── From shipping_info_master
  └── Factory Location
        └── From shipping_info_master
```

All addresses are linked to the same Medusa Customer.

## Event Flow Example

### Complete Customer Setup
```
1. party_master/create
     └── Creates Customer + Default Address
     ↓
2. shipping_info_master/create (first location)
     └── Adds Address 1 to Customer
     ↓
3. shipping_info_master/create (second location)
     └── Adds Address 2 to Customer
     ↓
4. shipping_info_master/create (third location)
     └── Adds Address 3 to Customer
```

## Address Usage in Orders

When a customer places an order:
- They can select any of their addresses
- Address is copied to order
- Shipping info is preserved
- Original customer address remains unchanged

## GST Number Handling

The `gst_no` field from Shipping Info:
- Stored in Shipping Info Master
- Not directly mapped to Medusa Address
- Can be used for:
  - Invoice generation
  - Tax calculations
  - Compliance reporting

## Related Events

- `party_master`: Parent customer entity
- `create-customer`: Reverse flow
- `customer-updated`: Customer updates

## Best Practices

1. **Create Party First**: Always ensure customer exists
2. **Verify GST Numbers**: Validate format if possible
3. **Standardize Addresses**: Use consistent formatting
4. **Multiple Locations**: Support business with many locations
5. **State Codes**: Ensure state names match conversion table

## Address Validation

While the plugin doesn't validate addresses, best practices include:
- Valid PIN codes (6 digits for India)
- Recognizable state names
- Complete address information
- Working phone numbers
