# Entity Relationship Diagram (ERD)

## 1. Overview

This document presents the Entity-Relationship Diagram (ERD) for the **AI Tech Marketplace** PostgreSQL transactional database, reflecting the relational schema defined in [database/schema.md](../database/schema.md).

---

## 2. Relational Model Diagram

```mermaid
erDiagram
    USERS ||--o{ USER_ROLES : "assigned"
    ROLES ||--o{ USER_ROLES : "defines"
    USERS ||--o{ ADDRESSES : "owns"
    USERS ||--o| CARTS : "maintains"
    USERS ||--o{ ORDERS : "places"
    USERS ||--o{ PRODUCTS : "sells"
    USERS ||--o{ REVIEWS : "writes"
    USERS ||--o{ CONVERSATIONS : "participates_in"

    CARTS ||--o{ CART_ITEMS : "contains"
    PRODUCTS ||--o{ CART_ITEMS : "added_as"

    CATEGORIES ||--o{ PRODUCTS : "classifies"
    BRANDS ||--o{ PRODUCTS : "brands"
    PRODUCTS ||--o{ PRODUCT_IMAGES : "has_gallery"
    PRODUCTS ||--|| INVENTORY : "tracks_stock"
    PRODUCTS ||--o{ REVIEWS : "reviewed_in"

    ORDERS ||--|{ ORDER_ITEMS : "contains"
    ORDERS ||--o{ PAYMENTS : "paid_by"
    PRODUCTS ||--o{ ORDER_ITEMS : "historical_reference"

    CONVERSATIONS ||--|{ CONVERSATION_MESSAGES : "contains"

    USERS {
        bigint id PK
        varchar email UK
        varchar password_hash
        varchar full_name
        varchar phone
        varchar status
        timestamp created_at
        timestamp updated_at
    }

    ROLES {
        bigint id PK
        varchar name UK
    }

    USER_ROLES {
        bigint user_id PK,FK
        bigint role_id PK,FK
    }

    CATEGORIES {
        bigint id PK
        varchar name
        varchar slug UK
        varchar status
        timestamp created_at
        timestamp updated_at
    }

    BRANDS {
        bigint id PK
        varchar name UK
        varchar slug UK
        varchar status
        timestamp created_at
        timestamp updated_at
    }

    PRODUCTS {
        bigint id PK
        bigint seller_id FK
        varchar sku UK
        varchar name
        varchar slug UK
        text description
        numeric price
        bigint category_id FK
        bigint brand_id FK
        jsonb specifications
        varchar status
        timestamp created_at
        timestamp updated_at
    }

    PRODUCT_IMAGES {
        bigint id PK
        bigint product_id FK
        text image_url
        int sort_order
        boolean is_primary
    }

    INVENTORY {
        bigint id PK
        bigint product_id UK,FK
        int quantity
        int reserved_quantity
        timestamp updated_at
    }

    ADDRESSES {
        bigint id PK
        bigint user_id FK
        varchar recipient_name
        varchar phone
        varchar address_line
        varchar ward
        varchar district
        varchar city
        varchar country
        boolean is_default
        timestamp created_at
        timestamp updated_at
    }

    CARTS {
        bigint id PK
        bigint user_id UK,FK
        timestamp created_at
        timestamp updated_at
    }

    CART_ITEMS {
        bigint id PK
        bigint cart_id FK
        bigint product_id FK
        int quantity
        timestamp created_at
        timestamp updated_at
    }

    ORDERS {
        bigint id PK
        varchar order_number UK
        bigint user_id FK
        varchar status
        varchar payment_method
        varchar payment_status
        numeric subtotal
        numeric shipping_fee
        numeric discount_amount
        numeric total_amount
        varchar shipping_recipient_name
        varchar shipping_phone
        text shipping_address
        timestamp created_at
        timestamp updated_at
    }

    ORDER_ITEMS {
        bigint id PK
        bigint order_id FK
        bigint product_id FK
        varchar product_name
        varchar sku
        int quantity
        numeric unit_price
        numeric subtotal
    }

    PAYMENTS {
        bigint id PK
        bigint order_id FK
        varchar payment_method
        varchar transaction_reference
        numeric amount
        varchar status
        timestamp created_at
        timestamp updated_at
    }

    REVIEWS {
        bigint id PK
        bigint user_id FK
        bigint product_id FK
        bigint order_item_id FK
        int rating
        text comment
        boolean is_verified_purchase
        timestamp created_at
        timestamp updated_at
    }

    CONVERSATIONS {
        uuid id PK
        bigint user_id FK
        timestamp created_at
        timestamp updated_at
    }

    CONVERSATION_MESSAGES {
        bigint id PK
        uuid conversation_id FK
        varchar role
        text content
        timestamp created_at
    }
```

---

## 3. Entity Classification & Cardinalities

| Entity Group | Entities | Cardinality & Integrity Rules |
| --- | --- | --- |
| **Identity & Access** | `USERS`, `ROLES`, `USER_ROLES`, `ADDRESSES` | - Users have a Many-to-Many relationship with Roles via `USER_ROLES`.<br/>- A User has Zero-to-Many `ADDRESSES`; at most one address is marked `is_default`. |
| **Catalog & Products** | `CATEGORIES`, `BRANDS`, `PRODUCTS`, `PRODUCT_IMAGES` | - Each Product belongs to one Seller (`USERS`), one Category and optionally one Brand.<br/>- Flexible product specifications are indexed via PostgreSQL `JSONB`.<br/>- `PRODUCT_IMAGES` provides a 1-to-N sorted image gallery. |
| **Inventory** | `INVENTORY` | - One-to-One with `PRODUCTS`. Available quantity is derived: `quantity - reserved_quantity`. |
| **Cart & Ordering** | `CARTS`, `CART_ITEMS`, `ORDERS`, `ORDER_ITEMS`, `PAYMENTS` | - A customer has exactly one active `CARTS`.<br/>- Unique composite constraint on `(cart_id, product_id)`.<br/>- `ORDERS` and `ORDER_ITEMS` capture immutable historical snapshots of recipient contact, shipping address, and product unit prices at purchase time.<br/>- `PAYMENTS` tracks gateway transaction states linked 1-to-N with `ORDERS`. |
| **Reviews & Feedback** | `REVIEWS` | - Customers submit 1-to-N verified reviews for purchased `PRODUCTS` with 1-5 star ratings. |
| **AI Conversations** | `CONVERSATIONS`, `CONVERSATION_MESSAGES` | - AI chat history linked to `user_id` using UUID keys for stateless session tracking. Messages store conversational roles (`USER`, `ASSISTANT`, `SYSTEM`). |
