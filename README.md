# E-commerce Database Design: A Senior SQL Developer's Blueprint

This design models a typical transactional e-commerce workflow: customer accounts, catalog browsing, inventory, cart checkout, orders, payments, fulfillment, returns, promotions, and auditability.

The goal is not to cram everything into one giant `orders` table. A durable e-commerce schema separates mutable shopping activity from immutable commercial records, keeps financial events auditable, and treats inventory as a ledger instead of a single fragile number.

## Core Design Principles

- **Separate cart from order**: carts are temporary and mutable; orders are legal/commercial records.
- **Snapshot purchased data**: order items store product name, SKU, and price at purchase time so historical orders do not change when the catalog changes.
- **Use ledgers for money and inventory**: payments, refunds, and stock movements should be append-friendly and auditable.
- **Normalize operational data, denormalize historical facts**: product catalog stays normalized; order records preserve transaction-time facts.
- **Design for state transitions**: order, payment, shipment, and return statuses should represent workflow states, not arbitrary text blobs.

## Entity Relationship Diagram

```mermaid
erDiagram
    CUSTOMER ||--o{ ADDRESS : has
    CUSTOMER ||--o{ CART : owns
    CUSTOMER ||--o{ SALES_ORDER : places
    CUSTOMER ||--o{ REVIEW : writes

    CATEGORY ||--o{ CATEGORY : parent_of
    CATEGORY ||--o{ PRODUCT_CATEGORY : groups
    PRODUCT ||--o{ PRODUCT_CATEGORY : belongs_to
    PRODUCT ||--o{ PRODUCT_VARIANT : has
    PRODUCT ||--o{ REVIEW : receives

    PRODUCT_VARIANT ||--o{ INVENTORY_BALANCE : stocked_at
    WAREHOUSE ||--o{ INVENTORY_BALANCE : stores
    PRODUCT_VARIANT ||--o{ INVENTORY_MOVEMENT : moves
    WAREHOUSE ||--o{ INVENTORY_MOVEMENT : records

    CART ||--o{ CART_ITEM : contains
    PRODUCT_VARIANT ||--o{ CART_ITEM : selected_as

    SALES_ORDER ||--o{ ORDER_ITEM : contains
    PRODUCT_VARIANT ||--o{ ORDER_ITEM : purchased_as
    SALES_ORDER ||--o{ PAYMENT : paid_by
    SALES_ORDER ||--o{ SHIPMENT : fulfilled_by
    SALES_ORDER ||--o{ ORDER_DISCOUNT : discounted_by
    PROMOTION ||--o{ ORDER_DISCOUNT : applied_as

    SHIPMENT ||--o{ SHIPMENT_ITEM : ships
    ORDER_ITEM ||--o{ SHIPMENT_ITEM : fulfilled_from

    SALES_ORDER ||--o{ RETURN_REQUEST : may_have
    RETURN_REQUEST ||--o{ RETURN_ITEM : contains
    ORDER_ITEM ||--o{ RETURN_ITEM : returned_from
    PAYMENT ||--o{ REFUND : refunded_by
    RETURN_REQUEST ||--o{ REFUND : may_trigger

    CUSTOMER {
        bigint customer_id PK
        varchar email UK
        varchar password_hash
        varchar first_name
        varchar last_name
        varchar phone
        varchar status
        timestamptz created_at
        timestamptz updated_at
    }

    ADDRESS {
        bigint address_id PK
        bigint customer_id FK
        varchar address_type
        varchar recipient_name
        varchar line1
        varchar line2
        varchar city
        varchar region
        varchar postal_code
        varchar country_code
        boolean is_default
    }

    PRODUCT {
        bigint product_id PK
        varchar title
        text description
        varchar brand
        varchar status
        timestamptz created_at
        timestamptz updated_at
    }

    PRODUCT_VARIANT {
        bigint variant_id PK
        bigint product_id FK
        varchar sku UK
        varchar barcode
        varchar option_summary
        numeric list_price
        numeric sale_price
        numeric weight
        varchar status
    }
