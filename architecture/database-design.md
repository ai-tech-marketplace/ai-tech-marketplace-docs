# Database Design

## 1. Database Technology

The primary transactional database is:

```text
PostgreSQL
```

The database is owned and accessed by the backend service.

Other services must not directly modify transactional data.

## 2. Database Principles

The database design follows these principles:

* Normalize transactional data where appropriate
* Use foreign keys for data integrity
* Use indexes for frequently queried fields
* Store timestamps for important entities
* Use UUID or BIGINT consistently
* Use database migrations
* Keep transactional data in PostgreSQL
* Avoid storing derived data as the source of truth

## 3. Core Entities

The initial database contains:

```text
User
Role
Category
Brand
Product
Inventory
Address
Cart
CartItem
Order
OrderItem
```

## 4. Entity Relationship

```text
                         ┌──────────────┐
                         │     Role     │
                         └───────┬──────┘
                                 │
                                 │
                         ┌───────▼──────┐
                         │     User     │
                         └───┬────┬─────┘
                             │    │
                 ┌───────────┘    └────────────┐
                 │                             │
                 ▼                             ▼
           ┌──────────┐                  ┌──────────┐
           │ Address  │                  │   Cart   │
           └──────────┘                  └────┬─────┘
                                              │
                                              ▼
                                        ┌───────────┐
                                        │ CartItem  │
                                        └─────┬─────┘
                                              │
                                              ▼
                                         ┌─────────┐
                                         │ Product │
                                         └────┬────┘
                                              │
                              ┌───────────────┼───────────────┐
                              │               │               │
                              ▼               ▼               ▼
                         Category          Brand         Inventory

User
 │
 └───────────────┐
                 ▼
              Order
                 │
                 ▼
             OrderItem
                 │
                 ▼
              Product
```

## 5. User

### Purpose

Stores application users.

### Main Fields

```text
User
-----------------------------
id
email
password_hash
full_name
phone
status
role_id
created_at
updated_at
```

### Constraints

* Email must be unique
* Password must never be stored as plain text
* Role is required
* Status is required

## 6. Role

### Purpose

Stores authorization roles.

### Main Fields

```text
Role
-----------------------------
id
name
created_at
```

Initial roles:

```text
CUSTOMER
SELLER
ADMIN
```

## 7. Category

### Purpose

Stores product categories.

### Main Fields

```text
Category
-----------------------------
id
name
slug
description
status
created_at
updated_at
```

Examples:

```text
Laptop
Smartphone
Monitor
PC Component
Accessory
```

## 8. Brand

### Purpose

Stores product brands.

### Main Fields

```text
Brand
-----------------------------
id
name
slug
description
created_at
updated_at
```

Examples:

```text
Apple
Dell
Lenovo
ASUS
MSI
Samsung
```

## 9. Product

### Purpose

Stores marketplace products.

### Main Fields

```text
Product
-----------------------------
id
sku
name
slug
description
price
category_id
brand_id
specifications
status
created_at
updated_at
```

### Specifications

Product specifications may differ between categories.

Therefore, the initial design may use PostgreSQL JSONB.

Example:

```json
{
  "cpu": "Intel Core Ultra 9",
  "gpu": "RTX 5070",
  "ram": "32GB",
  "storage": "1TB SSD",
  "display": "16 inch OLED"
}
```

### Constraints

* SKU must be unique
* Name is required
* Price must be non-negative
* Category is required
* Brand may be optional depending on product type

## 10. Inventory

### Purpose

Stores inventory information.

### Main Fields

```text
Inventory
-----------------------------
id
product_id
total_quantity
reserved_quantity
created_at
updated_at
```

Available quantity:

```text
available_quantity =
    total_quantity - reserved_quantity
```

The available quantity should not be stored as the source of truth if it can be safely derived.

## 11. Address

### Purpose

Stores customer delivery addresses.

### Main Fields

```text
Address
-----------------------------
id
user_id
recipient_name
phone
address_line
ward
district
city
country
is_default
created_at
updated_at
```

## 12. Cart

### Purpose

Represents a customer's active shopping cart.

### Main Fields

```text
Cart
-----------------------------
id
user_id
created_at
updated_at
```

A customer should normally have one active cart.

## 13. CartItem

### Purpose

Stores products in a cart.

### Main Fields

```text
CartItem
-----------------------------
id
cart_id
product_id
quantity
created_at
updated_at
```

Constraints:

* Quantity > 0
* A product should not appear multiple times in the same cart

A unique constraint should exist on:

```text
(cart_id, product_id)
```

## 14. Order

### Purpose

Represents a customer order.

### Main Fields

```text
Order
-----------------------------
id
order_number
user_id
status
total_amount
shipping_address_snapshot
created_at
updated_at
```

### Order Status

```text
PENDING
CONFIRMED
PROCESSING
SHIPPED
DELIVERED
CANCELLED
```

## 15. OrderItem

### Purpose

Stores products belonging to an order.

### Main Fields

```text
OrderItem
-----------------------------
id
order_id
product_id
product_name
sku
quantity
unit_price
subtotal
```

The item stores product information at the time of purchase.

This protects historical order data from future product changes.

Example:

```text
Product current price:
25,000,000 VND

Historical order:
unit_price = 23,000,000 VND
```

## 16. Relationships

### User → Role

```text
Role 1 ---- N User
```

### User → Address

```text
User 1 ---- N Address
```

### User → Cart

```text
User 1 ---- 1 Active Cart
```

### Cart → CartItem

```text
Cart 1 ---- N CartItem
```

### Product → CartItem

```text
Product 1 ---- N CartItem
```

### Category → Product

```text
Category 1 ---- N Product
```

### Brand → Product

```text
Brand 1 ---- N Product
```

### Product → Inventory

```text
Product 1 ---- 1 Inventory
```

### User → Order

```text
User 1 ---- N Order
```

### Order → OrderItem

```text
Order 1 ---- N OrderItem
```

### Product → OrderItem

```text
Product 1 ---- N OrderItem
```

## 17. Index Strategy

Important indexes include:

### User

```text
UNIQUE(email)
```

### Product

```text
UNIQUE(sku)
INDEX(category_id)
INDEX(brand_id)
INDEX(status)
INDEX(price)
```

### CartItem

```text
UNIQUE(cart_id, product_id)
```

### Order

```text
INDEX(user_id)
INDEX(status)
INDEX(created_at)
UNIQUE(order_number)
```

### OrderItem

```text
INDEX(order_id)
INDEX(product_id)
```

## 18. Audit Fields

Major entities should include:

```text
created_at
updated_at
```

Additional audit fields may be added later:

```text
created_by
updated_by
```

## 19. Database Migration

The backend will use database migrations.

Preferred technology:

```text
Flyway
```

Migration files will be stored in:

```text
ai-tech-marketplace-backend
```

Example:

```text
V1__create_roles.sql
V2__create_users.sql
V3__create_categories.sql
V4__create_brands.sql
V5__create_products.sql
V6__create_inventory.sql
V7__create_carts.sql
V8__create_orders.sql
```

## 20. Ownership

PostgreSQL transactional data is owned by the backend.

```text
Backend
   |
   v
PostgreSQL
```

The AI service must not directly modify marketplace transactional tables.

The AI service should obtain product information through controlled APIs or dedicated read/search mechanisms.
