---
title: Party Master Module
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# Party Master Module

## Overview

The Party Master module manages customer, supplier, and business party information synchronized from the ERP system. It serves as the primary source for customer data and maps to Medusa's Customer entities.

## Model: PartyMaster

### Database Table
`party_master`

### Fields

| Field | Type | Required | Indexed | Description |
|-------|------|----------|---------|-------------|
| `id` | ID | Yes | Yes (PK) | Primary key - auto-generated |
| `party_no` | Text | Yes | Yes | Party number from ERP |
| `party_code` | Text | Yes | Yes | Party code/identifier |
| `firm_name` | Text | Yes | No | Company/Firm name |
| `legal_name` | Text | Yes | No | Legal business name |
| `firm_add1` | Text | Yes | No | Address line 1 |
| `firm_add2` | Text | Yes | No | Address line 2 |
| `firm_add3` | Text | Yes | No | Address line 3 |
| `firm_pin_code` | Text | Yes | No | Postal/ZIP code |
| `firm_email` | Text | Yes | No | Business email address |
| `owner_mobile` | Text | Yes | No | Primary contact mobile number |
| `firm_tele` | Text | Yes | No | Landline telephone number |
| `whatsapp_no` | Text | Yes | No | WhatsApp contact number |
| `country_id` | Number | Yes | No | Country ID reference |
| `country` | Text | Yes | No | Country name |
| `state_id` | Number | Yes | No | State ID reference |
| `state` | Text | Yes | No | State name |
| `firm_city` | Text | Yes | No | City name |
| `firm_birth_date` | Text | No | No | Firm establishment date |
| `firm_anniversary_date` | Text | No | No | Firm anniversary date |
| `branch_no` | Text | Yes | No | Branch number |
| `branch_name` | Text | Yes | No | Branch name |
| `location_id` | Text | Yes | No | Location identifier |
| `location_name` | Text | Yes | No | Location name |
| `aadhaar_no` | Text | Yes | No | Aadhaar number (Indian ID) |
| `firm_pan` | Text | Yes | No | PAN card number |
| `gst_no` | Text | Yes | No | GST registration number |
| `lut_no` | Text | Yes | No | LUT (Letter of Undertaking) number |
| `identity_proof` | Text | Yes | No | Identity proof document |
| `cst_no` | Text | Yes | No | CST registration number |
| `lst_no` | Text | Yes | No | LST registration number |
| `service_tax_no` | Text | Yes | No | Service tax number |
| `bank_acc_no` | Text | Yes | No | Bank account number |
| `bank_name` | Text | Yes | No | Bank name |
| `bank_add1` | Text | Yes | No | Bank address line 1 |
| `bank_add2` | Text | Yes | No | Bank address line 2 |
| `bank_add3` | Text | Yes | No | Bank address line 3 |
| `bank_pin_code` | Text | Yes | No | Bank PIN code |
| `bank_ifsccode` | Text | Yes | No | Bank IFSC code |
| `bank_swiftcode` | Text | Yes | No | Bank SWIFT code |
| `is_customer` | Boolean | Yes | No | Flag: Is a customer |
| `is_supplier` | Boolean | Yes | No | Flag: Is a supplier |
| `is_account` | Boolean | Yes | No | Flag: Is an account |
| `is_salesman` | Boolean | Yes | No | Flag: Is a salesman |
| `is_factory` | Boolean | Yes | No | Flag: Is a factory |
| `factory_type` | Boolean | Yes | No | Factory type flag |
| `is_tools` | Boolean | Yes | No | Flag: Is tools related |
| `is_location` | Boolean | Yes | No | Flag: Is a location |
| `is_pd_head` | Number | Yes | No | PD head flag (0/1) |
| `is_inline_branch` | Boolean | Yes | No | Flag: Is inline branch |
| `is_scheme_member` | Boolean | Yes | No | Flag: Is scheme member |
| `control_acc_no` | Text | Yes | No | Control account number |
| `is_main_client` | Boolean | Yes | No | Flag: Is main client |
| `mapping_id` | Text | No | Yes | Medusa Customer ID mapping |

## Relationships

- **One-to-Many** with `shipping_info_master` (via `party_no`)
- **Many-to-One** with Medusa Customer (via `mapping_id`)

## Medusa Mapping

When a Party Master record is created/updated:

### Creates:
1. **Medusa Customer** with:
   - `first_name` / `last_name`: Derived from `firm_name`
   - `email`: Generated from `owner_mobile` (format: `{mobile}@gati-medusa.com`)
   - `phone`: Formatted Indian phone number
   - `has_account`: `true`
   - `addresses`: Default shipping/billing address from party data
   - `metadata`: Contains ERP `party_no` as `external_id`

2. **Auth Identity** with:
   - `provider`: `passwordless`
   - `entity_id`: Formatted phone number
   - `app_metadata.customer_id`: Medusa customer ID

3. **Customer Addresses** with:
   - Address details from party master
   - Default shipping/billing flags

### Updates:
- Customer name and metadata
- Customer addresses (matching by address data)

### Deletes:
- Medusa Customer
- Associated Customer Addresses

## Usage

### Service Methods

```typescript
// Get party master service
const partyMasterService = container.resolve(PARTY_MASTER_MODULE);

// Create party master
const party = await partyMasterService.createPartyMasters([{
  party_no: "12345",
  party_code: "CUST001",
  firm_name: "ABC Jewelers",
  // ... other fields
}]);

// Update party master
await partyMasterService.updatePartyMasters([{
  id: party[0].id,
  firm_name: "ABC Jewelers Pvt Ltd",
  mapping_id: "cus_123456789"
}]);

// List party masters
const parties = await partyMasterService.listPartyMasters({
  is_customer: true
});
```

## Webhook Event

**Event Name:** `party_master`

**Operations:**
- `create`: Creates new Medusa customer and auth identity
- `update`: Updates existing customer data
- `delete`: Removes customer and addresses

See [Webhook Events - Party Master](../webhook-events/party-master.md) for detailed flow.
