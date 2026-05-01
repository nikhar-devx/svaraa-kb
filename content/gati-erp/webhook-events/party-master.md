---
title: Party Master Webhook Events
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# Party Master Webhook Events

## Event Name
`party_master`

## Overview
Party Master webhook events handle the synchronization of customer and party data from the ERP system to Medusa. When a party is created, updated, or deleted in the ERP, these events ensure corresponding Medusa Customer entities are maintained.

## Event Payload Structure

```typescript
{
  operation: "create" | "update" | "delete",
  data: PartyMasterData[]
}
```

## Operations

### 1. CREATE Operation

**When**: New party added to ERP

**Flow**:
```
ERP Party Created
  └── Webhook Sent (party_master/create)
        └── Subscriber Receives Event
              ├── Query Party Master Record
              ├── Create Medusa Customer
              │     ├── Generate Email from Mobile
              │     ├── Format Phone Number
              │     ├── Split Firm Name (First/Last)
              │     └── Create Default Address
              ├── Create Auth Identity (Passwordless)
              └── Update Party Master with mapping_id
```

**Creates**:
- **Medusa Customer**:
  - `first_name`: First word of firm_name
  - `last_name`: Remaining words of firm_name
  - `email`: `{owner_mobile}@gati-medusa.com`
  - `phone`: Formatted Indian phone number
  - `has_account`: true
  - `metadata`: `{ external_id: party_no }`
  
- **Customer Address**:
  - Address lines from firm_add1/add2/add3
  - City, state, pin code
  - Default shipping and billing flags
  
- **Auth Identity**:
  - Provider: `passwordless`
  - Entity ID: Formatted phone number
  - App Metadata: `{ customer_id: customer.id }`

- **Customer Additional Data**:
  - GST number
  - PAN number
  - Birth date
  - Anniversary date
  - External ID (party_no)

**Error Handling**:
- Duplicate email: Auto-generated unique email
- Invalid phone: Formatting applied
- Missing address: Uses empty strings

### 2. UPDATE Operation

**When**: Existing party modified in ERP

**Flow**:
```
ERP Party Updated
  └── Webhook Sent (party_master/update)
        └── Subscriber Receives Event
              ├── Query Party Master (get mapping_id)
              ├── Check mapping_id exists
              ├── Update Medusa Customer
              │     ├── Update Name
              │     └── Update Metadata
              ├── Find/Update Customer Addresses
              │     ├── Match by Address Data
              │     ├── Update if Found
              │     └── Create if Not Found
              └── Log Update
```

**Updates**:
- **Customer Name**: `first_name` and `last_name` from `firm_name`
- **Customer Metadata**:
  - Email
  - Birth date
  - Anniversary date
  - GST number
  - PAN number

**Address Handling**:
- Searches for existing address by matching criteria
- If found: Updates the address
- If not found: Creates new address

**Important**: Updates only occur if `mapping_id` exists. If not mapped, update is skipped.

### 3. DELETE Operation

**When**: Party deleted from ERP

**Flow**:
```
ERP Party Deleted
  └── Webhook Sent (party_master/delete)
        └── Subscriber Receives Event
              ├── Check mapping_id exists
              ├── List Customer Addresses
              ├── Delete Customer
              └── Delete All Addresses
```

**Deletes**:
- **Medusa Customer** (by mapping_id)
- **All Customer Addresses** (associated with customer)
- **Auth Identity** (cascade deletion)

**Important**: 
- Only deletes if `mapping_id` exists
- Shipping info addresses (children) are also deleted via cascade

## Data Transformation

### Phone Number Formatting
- Input: Any format mobile number
- Output: Indian format with +91 prefix
- Example: `9876543210` → `+919876543210`

### Email Generation
- Format: `{mobile}@gati-medusa.com`
- Purpose: Unique email for each customer
- Example: `9876543210@gati-medusa.com`

### Name Splitting
- Input: `ABC Jewelers Pvt Ltd`
- First Name: `ABC`
- Last Name: `Jewelers Pvt Ltd`

### State Code Extraction
- Input: `Maharashtra [MH]`
- Output: `MH`
- Used for address province field

## Address Formatting

Addresses are formatted for Medusa compatibility:
```typescript
{
  address_1: firm_add1 || "",
  address_2: firm_add2 || firm_add3 || "",
  city: firm_city,
  province: extracted_state_code,
  postal_code: firm_pin_code,
  country_code: country === "India" ? "in" : country_code
}
```

## Mapping ID Strategy

1. **Initial Creation**:
   - Party Master created without `mapping_id`
   - Customer created in Medusa
   - `mapping_id` updated with Customer ID

2. **Subsequent Operations**:
   - All operations use `mapping_id` to locate Medusa entity
   - If `mapping_id` is null, operation is skipped

3. **One-way Mapping**:
   - ERP → Medusa: Via `mapping_id`
   - Medusa → ERP: Via `metadata.external_id` (party_no)

## Error Scenarios

### No Mapping ID
```
Update/Delete without mapping_id
  └── Log Warning
        └── Skip Operation
```

### Duplicate Phone
```
Create with existing phone
  └── Generate Unique Email
        └── Append Timestamp or Random String
```

### Invalid Address Data
```
Missing Address Fields
  └── Use Empty Strings
        └── Create Address with Available Data
```

## Event Logging

All operations are logged via `Logger`:
- Operation start
- Mapping ID status
- Success/failure status
- Error details

## Related Events

- `shipping_info_master`: Creates addresses for party
- `create-customer`: Reverse flow (Medusa → ERP)
- `customer-updated`: Reverse flow updates
