# Database Schema

## 1. Database

Primary database:

```text
PostgreSQL
```

The database stores transactional marketplace data.

---

## 2. Main Entities

The initial database contains:

```text
users
roles
user_roles

categories
brands
products
product_images

carts
cart_items

addresses

orders
order_items

inventory

conversations
conversation_messages
```

---

## 3. Entity Relationship

```text
users
  |
  ├── user_roles ── roles
  |
  ├── addresses
  |
  ├── carts
  |     |
  |     └── cart_items ── products
  |
  ├── orders
  |     |
  |     └── order_items ── products
  |
  └── conversations
          |
          └── conversation_messages


categories ── products ── brands
                  |
                  └── product_images

products ── inventory
```

---

## 4. users

Stores customer, seller and administrator accounts.

| Column        | Type         | Constraints      |
| ------------- | ------------ | ---------------- |
| id            | BIGSERIAL    | PK               |
| email         | VARCHAR(255) | UNIQUE, NOT NULL |
| password_hash | VARCHAR(255) | NOT NULL         |
| full_name     | VARCHAR(100) | NOT NULL         |
| phone         | VARCHAR(30)  | NULL             |
| status        | VARCHAR(30)  | NOT NULL         |
| created_at    | TIMESTAMP    | NOT NULL         |
| updated_at    | TIMESTAMP    | NOT NULL         |

Possible status:

```text
ACTIVE
INACTIVE
LOCKED
```

---

## 5. roles

Stores system roles.

| Column | Type        | Constraints      |
| ------ | ----------- | ---------------- |
| id     | BIGSERIAL   | PK               |
| name   | VARCHAR(50) | UNIQUE, NOT NULL |

Initial roles:

```text
CUSTOMER
SELLER
ADMIN
```

---

## 6. user_roles

Many-to-many relationship between users and roles.

| Column  | Type   | Constraints |
| ------- | ------ | ----------- |
| user_id | BIGINT | FK users.id |
| role_id | BIGINT | FK roles.id |

Primary key:

```text
(user_id, role_id)
```

---

## 7. categories

Product categories.

| Column     | Type         | Constraints      |
| ---------- | ------------ | ---------------- |
| id         | BIGSERIAL    | PK               |
| name       | VARCHAR(100) | NOT NULL         |
| slug       | VARCHAR(150) | UNIQUE, NOT NULL |
| status     | VARCHAR(30)  | NOT NULL         |
| created_at | TIMESTAMP    | NOT NULL         |
| updated_at | TIMESTAMP    | NOT NULL         |

Examples:

```text
Laptop
Smartphone
Monitor
Keyboard
Mouse
Headphone
Tablet
```

---

## 8. brands

Product brands.

| Column     | Type         | Constraints      |
| ---------- | ------------ | ---------------- |
| id         | BIGSERIAL    | PK               |
| name       | VARCHAR(100) | UNIQUE, NOT NULL |
| slug       | VARCHAR(150) | UNIQUE, NOT NULL |
| status     | VARCHAR(30)  | NOT NULL         |
| created_at | TIMESTAMP    | NOT NULL         |
| updated_at | TIMESTAMP    | NOT NULL         |

Examples:

```text
ASUS
Lenovo
Dell
HP
Apple
Samsung
Logitech
```

---

## 9. products

Main product table.

| Column         | Type          | Constraints      |
| -------------- | ------------- | ---------------- |
| id             | BIGSERIAL     | PK               |
| sku            | VARCHAR(100)  | UNIQUE, NOT NULL |
| name           | VARCHAR(255)  | NOT NULL         |
| slug           | VARCHAR(300)  | UNIQUE, NOT NULL |
| description    | TEXT          | NULL             |
| price          | NUMERIC(15,2) | NOT NULL         |
| category_id    | BIGINT        | FK               |
| brand_id       | BIGINT        | FK               |
| specifications | JSONB         | NULL             |
| status         | VARCHAR(30)   | NOT NULL         |
| created_at     | TIMESTAMP     | NOT NULL         |
| updated_at     | TIMESTAMP     | NOT NULL         |

Example `specifications`:

```json
{
  "cpu": "Intel Core Ultra 9",
  "gpu": "RTX 5070",
  "ram": "32GB",
  "storage": "1TB SSD",
  "display": "16 inch OLED"
}
```

---

## 10. product_images

Stores product images.

| Column     | Type      | Constraints |
| ---------- | --------- | ----------- |
| id         | BIGSERIAL | PK          |
| product_id | BIGINT    | FK          |
| image_url  | TEXT      | NOT NULL    |
| sort_order | INTEGER   | NOT NULL    |
| is_primary | BOOLEAN   | NOT NULL    |

---

## 11. inventory

Stores current inventory.

| Column            | Type      | Constraints |
| ----------------- | --------- | ----------- |
| id                | BIGSERIAL | PK          |
| product_id        | BIGINT    | UNIQUE, FK  |
| quantity          | INTEGER   | NOT NULL    |
| reserved_quantity | INTEGER   | NOT NULL    |
| updated_at        | TIMESTAMP | NOT NULL    |

Available inventory:

```text
available = quantity - reserved_quantity
```

---

## 12. addresses

Customer shipping addresses.

| Column         | Type         | Constraints |
| -------------- | ------------ | ----------- |
| id             | BIGSERIAL    | PK          |
| user_id        | BIGINT       | FK          |
| recipient_name | VARCHAR(100) | NOT NULL    |
| phone          | VARCHAR(30)  | NOT NULL    |
| address_line   | VARCHAR(255) | NOT NULL    |
| ward           | VARCHAR(100) | NULL        |
| district       | VARCHAR(100) | NULL        |
| city           | VARCHAR(100) | NOT NULL    |
| country        | VARCHAR(100) | NOT NULL    |
| is_default     | BOOLEAN      | NOT NULL    |
| created_at     | TIMESTAMP    | NOT NULL    |
| updated_at     | TIMESTAMP    | NOT NULL    |

---

## 13. carts

One active cart per customer.

| Column     | Type      | Constraints |
| ---------- | --------- | ----------- |
| id         | BIGSERIAL | PK          |
| user_id    | BIGINT    | UNIQUE, FK  |
| created_at | TIMESTAMP | NOT NULL    |
| updated_at | TIMESTAMP | NOT NULL    |

---

## 14. cart_items

Products inside a cart.

| Column     | Type      | Constraints |
| ---------- | --------- | ----------- |
| id         | BIGSERIAL | PK          |
| cart_id    | BIGINT    | FK          |
| product_id | BIGINT    | FK          |
| quantity   | INTEGER   | NOT NULL    |
| created_at | TIMESTAMP | NOT NULL    |
| updated_at | TIMESTAMP | NOT NULL    |

Unique constraint:

```text
(cart_id, product_id)
```

---

## 15. orders

Customer orders.

| Column                  | Type          | Constraints      |
| ----------------------- | ------------- | ---------------- |
| id                      | BIGSERIAL     | PK               |
| order_number            | VARCHAR(50)   | UNIQUE, NOT NULL |
| user_id                 | BIGINT        | FK               |
| status                  | VARCHAR(30)   | NOT NULL         |
| total_amount            | NUMERIC(15,2) | NOT NULL         |
| shipping_recipient_name | VARCHAR(100)  | NOT NULL         |
| shipping_phone          | VARCHAR(30)   | NOT NULL         |
| shipping_address        | TEXT          | NOT NULL         |
| created_at              | TIMESTAMP     | NOT NULL         |
| updated_at              | TIMESTAMP     | NOT NULL         |

Shipping information is copied into the order so historical orders remain correct even if the user's address changes later.

---

## 16. order_items

Products purchased in an order.

| Column       | Type          | Constraints |
| ------------ | ------------- | ----------- |
| id           | BIGSERIAL     | PK          |
| order_id     | BIGINT        | FK          |
| product_id   | BIGINT        | FK          |
| product_name | VARCHAR(255)  | NOT NULL    |
| sku          | VARCHAR(100)  | NOT NULL    |
| quantity     | INTEGER       | NOT NULL    |
| unit_price   | NUMERIC(15,2) | NOT NULL    |
| subtotal     | NUMERIC(15,2) | NOT NULL    |

Historical product information is copied into the order item.

---

## 17. conversations

AI conversations.

| Column     | Type      | Constraints |
| ---------- | --------- | ----------- |
| id         | UUID      | PK          |
| user_id    | BIGINT    | FK          |
| created_at | TIMESTAMP | NOT NULL    |
| updated_at | TIMESTAMP | NOT NULL    |

---

## 18. conversation_messages

Messages inside AI conversations.

| Column          | Type        | Constraints |
| --------------- | ----------- | ----------- |
| id              | BIGSERIAL   | PK          |
| conversation_id | UUID        | FK          |
| role            | VARCHAR(30) | NOT NULL    |
| content         | TEXT        | NOT NULL    |
| created_at      | TIMESTAMP   | NOT NULL    |

Possible roles:

```text
USER
ASSISTANT
SYSTEM
```

---

## 19. Indexes

Initial indexes:

```text
users(email)

products(sku)
products(slug)
products(category_id)
products(brand_id)
products(status)

inventory(product_id)

orders(order_number)
orders(user_id)
orders(status)

cart_items(cart_id)

addresses(user_id)

conversation_messages(conversation_id)
```

Additional indexes should be introduced based on actual query patterns.

---

## 20. Database Principles

The database follows these principles:

1. PostgreSQL is the source of truth for transactional data.
2. Foreign keys should protect relational integrity.
3. Prices in orders are historical snapshots.
4. Product specifications use JSONB for flexibility.
5. Soft deletion should be preferred for important business entities.
6. Database migrations must be version controlled.
7. Application code must not manually modify production schemas.
8. Schema changes must go through migration files.
