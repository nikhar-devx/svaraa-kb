---
title: Plugin Gati Documentation
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# Plugin Gati Documentation

## Overview

Plugin Gati is a comprehensive ERP integration plugin for Medusa v2, designed to synchronize data between Gati ERP system and Medusa e-commerce platform. This plugin enables seamless data flow for jewelry and precious goods businesses, handling complex product structures, inventory management, and customer relationships.

## Version

Current Version: 0.0.34-beta.24

## Purpose

The plugin serves as a bridge between Gati ERP and Medusa, enabling:

- **Real-time synchronization** of master data (parties, categories, collections)
- **Product and variant management** with extended attributes for jewelry items
- **Inventory tracking** through inward master records
- **Customer management** with shipping address synchronization
- **Pricing and discount management** with markup calculations
- **Webhook event handling** for create, update, and delete operations

## Key Features

### 1. Master Data Synchronization
Synchronizes master data from ERP to Medusa:
- Party Master (Customers/Suppliers)
- Category Master & Category Group Master
- Collection Master & Collection Group Master
- Sub Category Master
- Shipping Info Master
- Raw Master, Quality Master, Tone Master, Shape Master, Item Size Master
- Discount Master
- Promocode Master

### 2. Product Management
Extended product and variant models for jewelry-specific attributes:
- Extended Product (Style Master)
- Extended Variant (Party Style Master)
- Inward Master (Inventory tracking)

### 3. Webhook Event System
Comprehensive event handling for ERP webhooks:
- Create, Update, Delete operations for all masters
- Automatic Medusa entity creation/mapping
- Bidirectional ID mapping between ERP and Medusa

### 4. Inventory Management
Advanced inventory tracking:
- Inward master records for stock receipts
- Location-based inventory tracking
- Status management (On Hand, Branch Issue, etc.)

## Architecture

### Module Structure
```
src/modules/
├── party-master/          # Customer/Party synchronization
├── category-master/       # Product categories
├── category-group-master/ # Category groups
├── collection-master/     # Product collections
├── collection-group-master/ # Collection groups
├── sub-category-master/   # Sub-categories
├── shipping-info-master/  # Customer shipping addresses
├── extended-product/      # Extended product (Style Master)
├── extended-variant/      # Extended variant (Party Style Master)
├── inward-master/         # Inventory inward records
├── raw-master/            # Raw material master
├── quality-master/        # Quality/diamond clarity master
├── tone-master/           # Tone/color master
├── shape-master/          # Shape master
├── item-size-master/      # Item size master
├── discount-master/       # Discount/markup rules
├── promocode-master/      # Promotional codes
├── erp/                   # ERP configuration
├── erp-event/             # ERP event logging
└── variant-option-sync-queue/ # Variant option sync queue
```

### Workflow Structure
```
src/workflows/
├── create-extended-product-from-product/  # Product creation workflow
├── inward-master/                         # Inward processing workflows
├── party-style-master/                    # Party style workflows
├── category-master/                       # Category sync workflows
├── category-group-master/                 # Category group workflows
├── collection-master/                     # Collection sync workflows
├── collection-group-master/               # Collection group workflows
├── sub-category-master/                   # Sub-category workflows
├── raw-master/                            # Raw master workflows
├── tone-master/                           # Tone master workflows
├── discount-master/                       # Discount workflows
├── custom-prices/                         # Custom pricing workflows
├── orders/                                # Order sync workflows
└── notification/                          # Notification workflows
```

### Subscriber Structure
```
src/subscribers/
├── party-master.ts           # Party master event handler
├── party-master-location.ts  # Party location handler
├── category-master.ts        # Category master event handler
├── category-group-master.ts  # Category group event handler
├── collection-master.ts      # Collection master event handler
├── sub-category-master.ts    # Sub-category event handler
├── shipping-info-master.ts   # Shipping info event handler
├── product-sync.ts           # Product synchronization handler
├── create-customer.ts        # Customer creation handler
├── cutomer-updated.ts        # Customer update handler
└── mark-inward.ts            # Inward marking handler
```

## Data Flow

### 1. ERP to Medusa (Webhook Events)
```
ERP System → Webhook → Subscriber → Workflow → Medusa Entity → ID Mapping
```

### 2. Product Creation Flow
```
ERP Style Master → Extended Product → Medusa Product + Extended Data → OpenSearch Sync
```

### 3. Inventory Flow
```
ERP Inward → Inward Master → Stock Update → Extended Variant Update → Inventory Module
```

### 4. Customer Flow
```
ERP Party Master → Party Master → Medusa Customer + Auth Identity → Shipping Info → Addresses
```

## Mapping Strategy

The plugin uses a **mapping_id** strategy to link ERP records with Medusa entities:

1. **ERP Record Creation**: ERP data is stored in plugin-specific tables
2. **Medusa Entity Creation**: Corresponding Medusa entities are created (products, customers, categories, etc.)
3. **ID Mapping**: Medusa entity IDs are stored in the `mapping_id` field of ERP records
4. **Update Operations**: Uses `mapping_id` to find and update existing Medusa entities
5. **Delete Operations**: Uses `mapping_id` to delete corresponding Medusa entities

## Webhook Events

All webhook events follow the same pattern:
- **Event Name**: `{module_name}` (e.g., `party_master`, `category_master`)
- **Operations**: `create`, `update`, `delete`
- **Payload**: Array of data objects

Example payload structure:
```json
{
  "operation": "create",
  "data": [
    {
      "id": "erp_record_id",
      "field1": "value1",
      "field2": "value2"
    }
  ]
}
```

## Dependencies

- **Medusa Framework**: 2.7.0
- **MikroORM**: 6.4.3 (Database ORM)
- **AWS SDK**: ECS client for deployment
- **Lodash**: Utility functions

## Configuration

The plugin requires ERP configuration stored in the `erp` module, including:
- ERP ID and name
- Webhook endpoints
- Authentication credentials

## Documentation Sections

- [Module Documentation](modules/) - Detailed field documentation for each module
- [Webhook Events](webhook-events/) - Event configuration and flow documentation

## Support

For issues and feature requests, please refer to the plugin repository or contact the development team.
