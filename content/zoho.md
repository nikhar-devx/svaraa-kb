---
title: Zoho CRM Integration
created: 2026-05-01
domain: backend
tags:
  - architecture
  - backend
---

# Zoho CRM Integration

Syncs customer touchpoints to Zoho CRM as contacts, tasks, and calendar events.

| Touchpoint         | Medusa event                                            | Zoho action                                                                     |
| ------------------ | ------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Customer registers | `extended_customer.created`                             | Create contact, set `Lead_Source: Customer Registration`                        |
| Form submitted     | `form-response`                                         | Create/find contact → update `Lead_Source` → task/event                         |
| Cart abandoned     | `cart.updated`                                          | Create/find contact → update `Lead_Source: Abandoned Cart` → create/update task |
| Profile updated    | `customer.updated`                                      | Update contact fields (name, email, phone, KYC)                                 |
| Address changed    | `customer-address.created` / `customer-address.updated` | Update mailing address fields on contact                                        |

---

## Flows

### Registration

```mermaid
flowchart TD
    E["extended_customer.created"] --> CC["createCRMContact<br/>{ Lead_Source: Customer Registration }"]
    CC --> Dup{DUPLICATE_DATA?}
    Dup -->|yes| D["whoId = duplicate_record.id"]
    Dup -->|no| N["whoId = details.id"]
    D --> Store["extended_customer.zoho_contact_id = whoId"]
    N --> Store
```

### Form Submission

```mermaid
flowchart TD
    E["form-response"] --> Op{operation}
    Op -->|create| Names{full_name or<br/>mobile_number?}
    Names -->|neither| Drop["skip"]
    Names -->|yes| CC["createCRMContact<br/>→ resolve whoId<br/>→ store on form_response"]
    CC --> UC["updateCRMContact<br/>{ Lead_Source: form type }"]
    UC --> Type{type in VIRTUAL_CALL<br/>STORE_VISIT<br/>BOOK_YOUR_TRIAL?}
    Type -->|yes| Ev["createCRMEvent<br/>(scheduled appointment)"]
    Type -->|no| Task["createCRMTask<br/>(follow-up task)"]
    Op -->|booking| BEv["createCRMEvent<br/>using form_response.whoId"]
```

### Abandoned Cart

```mermaid
flowchart TD
    E["cart.updated"] --> Guard{no customer / completed<br/>/ no items / POS_CART?}
    Guard -->|yes| Drop["skip"]
    Guard -->|no| R1{extended_customer<br/>.zoho_contact_id set?}
    R1 -->|yes| W1["whoId = zoho_contact_id"]
    R1 -->|no| R2{form_response<br/>by mobile?}
    R2 -->|yes| W2["whoId = form_response.whoId"]
    R2 -->|no| W3["createCRMContact<br/>→ store on extended_customer"]
    W1 --> UC["updateCRMContact<br/>{ Lead_Source: Abandoned Cart }"]
    W2 --> UC
    W3 --> UC
    UC --> TCheck{cart.metadata<br/>.zoho_task_id?}
    TCheck -->|yes| UT["updateCRMTask<br/>(refresh item URLs)"]
    TCheck -->|no| CT["createCRMTask<br/>→ store task ID in cart.metadata"]
```

### Profile / Address Update

```mermaid
flowchart TD
    E["customer.updated<br/>customer-address.created / .updated"] --> Check{zoho_contact_id<br/>set?}
    Check -->|yes| U["updateCRMContactData<br/>(name, email, phone, KYC, address)"]
    Check -->|no| C["createCRMContact<br/>→ store whoId"]
```

---

## Key Points

- **`zoho_contact_id`** on `extended_customer` — anchor for all Zoho writes; resolved once, reused forever.
- **`Lead_Source`** reflects the latest touchpoint; always overwritten on each action.
- **`updateCRMContact`** (Lead_Source only) and **`updateCRMContactData`** (profile fields only) are kept separate so profile edits don't overwrite the lead source.
- **DUPLICATE_DATA**: Zoho returns this when a contact already exists by email/mobile. Use `duplicate_record.id` as `whoId`.
- **Token**: single in-memory OAuth2 token, refreshed if last refresh was >2 min ago. Cleared on server restart.
