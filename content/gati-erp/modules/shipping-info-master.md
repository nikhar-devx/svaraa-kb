---
title: Shipping Info Master Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# Shipping Info Master Module

## Overview

The Shipping Info Master module manages customer shipping addresses synchronized from the ERP system. These are additional shipping locations for parties and map to Medusa's CustomerAddress entities.

## Model: ShippingInfoMaster

### Database Table
`shipping_info_master`

### Fields

| Field | Type | Required | Indexed | Description |
|-------|------|----------|---------|-------------|
| `id` | ID | Yes | Yes (PK) | Primary key - auto-generated |
| `party_no` | Text | Yes | Yes | Reference to Party Master (party_no) |
| `shipping_id` | Number | Yes | Yes | Shipping address ID from ERP |
| `shipping_code` | Text | Yes | No | Shipping address code |
| `shipping_name` | Text | Yes | No | Shipping location name |
| `mobile_no` | Text | Yes | No | Contact mobile number |
| `email` | Text | Yes | No | Contact email address |
| `address1` | Text | Yes | No | Address line 1 |
| `address2` | Text | Yes | No | Address line 2 |
| `city` | Text | Yes | No | City name |
| `state` | Text | Yes | No | State name |
| `country` | Text | Yes | No | Country name |
| `pin_code` | Text | Yes | No | Postal/ZIP code |
| `gst_no` | Text | Yes | No | GST number for this location |
| `mapping_id` | Text | No | Yes | Medusa CustomerAddress ID mapping |

## Relationships

- **Many-to-One** with Party Master (via `party_no`)
- **Many-to-One** with Medusa CustomerAddress (via `mapping_id`)

## Medusa Mapping

When a Shipping Info Master record is created/updated:

### Creates:
**Medusa CustomerAddress** with:
- `customer_id`: Mapped from Party Master's `mapping_id`
- `address_name`: `shipping_name`
- `company`: `shipping_name`
- `first_name`: Party's `firm_name`
- `last_name`: Party's `firm_name`
- `address_1`: `address1`
- `address_2`: `address2`
- `city`: `city`
- `province`: State code (converted from state name)
- `country_code`: `"in"` (India)
- `postal_code`: `pin_code`
- `is_default_shipping`: `false`
- `is_default_billing`: `false`
- `metadata`:
  - `external_id`: Shipping info master `id`

### Updates:
- Address fields (address lines, city, state, pin code)
- Company/address name

### Deletes:
- Medusa CustomerAddress (only if `mapping_id` exists)

## Usage

### Service Methods

```typescript
// Get shipping info master service
const shippingInfoMasterService = container.resolve(SHIPPING_INFO_MASTER_MODULE);

// Create shipping info master
const shippingInfo = await shippingInfoMasterService.createShippingInfoMasters([{
  party_no: "12345",
  shipping_id: 5001,
  shipping_code: "SH001",
  shipping_name: "Warehouse Location",
  mobile_no: "9876543210",
  email: "warehouse@example.com",
  address1: "123 Industrial Area",
  address2: "Phase 2",
  city: "Mumbai",
  state: "Maharashtra",
  country: "India",
  pin_code: "400001",
  gst_no: "27AABCU9603R1ZX"
}]);

// Update with mapping
await shippingInfoMasterService.updateShippingInfoMasters([{
  id: shippingInfo[0].id,
  address1: "456 New Industrial Area",
  mapping_id: "caddr_123456789"
}]);

// List shipping info for a party
const shippingAddresses = await shippingInfoMasterService.listShippingInfoMasters({
  party_no: "12345"
});
```

## Webhook Event

**Event Name:** `shipping_info_master`

**Operations:**
- `create`: Creates new customer address in Medusa
- `update`: Updates existing customer address
- `delete`: Removes customer address

See [Webhook Events - Shipping Info Master](../webhook-events/shipping-info-master.md) for detailed flow.

## State Code Mapping

The plugin includes state code conversion for Indian states. Common mappings include:
- Maharashtra → MH
- Gujarat → GJ
- Rajasthan → RJ
- Delhi → DL
- Karnataka → KA
- Tamil Nadu → TN
- West Bengal → WB
- And more...

## Multiple Addresses

A single Party Master can have multiple Shipping Info Master records:

```
Party Master (ABC Jewelers)
  ├── Default Address (from party master)
  ├── Warehouse Location (shipping_info_master)
  ├── Showroom Address (shipping_info_master)
  └── Factory Location (shipping_info_master)
```

All addresses are stored as CustomerAddress entities in Medusa under the same Customer.
