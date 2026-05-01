---
title: Product Sync Webhook Events
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
  - catalog
---

# Product Sync Webhook Events

## Event Name
`product-status-synced`

## Overview
Product Sync events handle the synchronization of product data from Medusa back to Extended Product records and trigger OpenSearch indexing. This is a reverse-flow event (Medusa → Plugin).

## Event Payload Structure

```typescript
{
  productId: string;
  additional_data: {
    extendedProductData: StyleDetail;
    styleDetails: StyleDetailLine[];
    styleExplorations: StyleExplorationLine[];
    styleCollections: StyleCollectionLine[];
  }
}
```

## Purpose

Product Sync events:
- **Create Extended Products**: When Medusa products are created
- **Update Status**: Sync product status to Extended Products
- **Index in OpenSearch**: Trigger search engine updates
- **Link Entities**: Connect Medusa products with ERP data

## Flow Direction

Unlike other webhook events (ERP → Medusa), this event flows:
```
Medusa Product Created/Updated
  └── Medusa Emits Event
        └── Plugin Subscriber Receives
              ├── Create/Update Extended Product
              ├── Update Extended Variants
              └── Sync to OpenSearch
```

## Event Trigger

This event is emitted by Medusa when:
- Products are created with additional_data
- Product status changes
- Product metadata is updated
- Variants are modified

## Operations

### 1. Extended Product Creation

**When**: Medusa product created with ERP data

**Flow**:
```
Medusa Product Created
  └── Event Emitted (product-status-synced)
        └── Subscriber Receives Event
              ├── Create Extended Product
              │     ├── Map Product Fields
              │     ├── Create Style Detail Lines
              │     ├── Create Style Exploration Lines
              │     ├── Create Style Collection Lines
              │     └── Create Remote Link
              └── Update Extended Variants
                    └── Sync Variant Status
```

**Creates**:
- **ExtendedProduct**: Full ERP data
- **StyleDetailsLine**: Material breakdown
- **StyleExplorationLine**: Variations
- **StyleCollectionLine**: Collection associations
- **RemoteLink**: Connects Product ↔ ExtendedProduct

### 2. Variant Status Update

**When**: Product variants are modified

**Flow**:
```
Variant Status Changed
  └── Event Triggered
        └── Update Extended Variant
              ├── Set Status (draft/published)
              ├── Update Sales Channels
              └── Emit OpenSearch Event
```

**Updates**:
- **ExtendedVariant.status**: draft/proposed/published/rejected
- **ExtendedVariant.sales_channels**: Channel assignments
- **OpenSearch Index**: Trigger reindex

### 3. OpenSearch Sync

**When**: Product data needs indexing

**Flow**:
```
Product Data Ready
  └── Emit sync-to-os-product Event
        └── OpenSearch Indexer Processes
              ├── Index Product Data
              ├── Index Variant Data
              └── Make Searchable
```

**Emits**:
```typescript
{
  eventName: "sync-to-os-product",
  data: {
    type: "product",
    operation: "create",
    id: productId
  }
}
```

## Data Transformation

### Product → Extended Product
Medusa product data is transformed to Extended Product:
```typescript
{
  style_id: additional_data.extendedProductData.style_id,
  style_code: additional_data.extendedProductData.style_code,
  // ... map all fields
  mapping_id: productId
}
```

### Variant → Extended Variant
Medusa variant status updates Extended Variant:
```typescript
{
  status: variant.status,  // draft → DRAFT, published → PUBLISHED
  sales_channels: variant.sales_channels
}
```

## Extended Product Creation Workflow

The `create-extended-product-from-product` workflow:

1. **Validate Input**: Check product and additional_data
2. **Create Extended Product**: Store ERP data
3. **Create Detail Lines**: Material specifications
4. **Create Exploration Lines**: Variations
5. **Create Collection Lines**: Collection associations
6. **Update Relationships**: Link lines to Extended Product
7. **Create Remote Link**: Connect to Medusa Product
8. **Emit Sync Event**: Trigger OpenSearch indexing

## Variant Status Workflow

The `update-extended-variant-status` workflow:

1. **Query Variants**: Get variants by product IDs
2. **Map Status**: Convert Medusa status to ExtendedVariant status
3. **Update Extended Variants**: Batch update status
4. **Emit OpenSearch Event**: Index updated variants

## Status Mapping

| Medusa Status | ExtendedVariant Status |
|---------------|------------------------|
| draft | DRAFT |
| proposed | PROPOSED |
| published | PUBLISHED |
| rejected | REJECTED |

## OpenSearch Integration

### Events Emitted
```typescript
// Product sync
{
  eventName: "sync-to-os-product",
  data: { type: "product", operation: "create", id: productId }
}

// Variant sync
{
  eventName: "sync-to-os-variant",
  data: { type: "variant", operation: "update", id: [variantIds] }
}
```

### Indexing Process
1. Receive sync event
2. Fetch product/variant data
3. Transform for OpenSearch
4. Update search index
5. Make searchable

## Error Handling

### Missing Additional Data
```
Product Created without ERP Data
  └── Log Warning
        └── Skip Extended Product Creation
```

### Failed OpenSearch Sync
```
OpenSearch Event Emitted
  └── Indexer Fails
        └── Log Error
              └── Retry Logic (if configured)
```

## Related Events

- `product.created`: Medusa product creation
- `product.updated`: Medusa product updates
- `sync-to-os-product`: OpenSearch product sync
- `sync-to-os-variant`: OpenSearch variant sync

## Best Practices

1. **Always Include Data**: Provide complete additional_data
2. **Validate Before Emit**: Check data integrity
3. **Monitor Sync Status**: Track OpenSearch indexing
4. **Handle Failures**: Implement retry logic
5. **Batch Updates**: Group variant updates for efficiency

## Performance Considerations

- Workflow runs asynchronously
- Batch updates for multiple variants
- OpenSearch indexing is non-blocking
- Event-driven architecture prevents bottlenecks

## Debugging

### Check Event Emission
```typescript
// Verify event is emitted
logger.info(`Product synced: ${productId}`);
```

### Monitor Workflow
```typescript
// Check workflow execution
const { result } = await workflow.run({ input });
logger.info(`Extended product created: ${result.extendedProduct.id}`);
```

### OpenSearch Verification
```typescript
// Verify indexing
const indexStatus = await checkOpenSearchIndex(productId);
```
