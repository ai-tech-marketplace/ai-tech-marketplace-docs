# Database Schema

## 1. Database

Primary database:

```text
PostgreSQL
```

The database stores transactional marketplace data.

---

## 2. Main Entities

The database contains:

```text
users
roles
user_roles

categories
brands
products
product_images

inventory

carts
cart_items

addresses

orders
order_items
payments

coupons
coupon_usages

commission_settings
seller_order_settlements

reviews

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
  |     ├── order_items ── products
  |     |     |
  |     |     └── seller_order_settlements (tied to seller)
  |     |
  |     └── payments
  |
  ├── seller_order_settlements (as seller)
  |
  ├── reviews ── products
  |
  ├── products (as seller)
  |
  └── conversations
          |
          └── conversation_messages


categories ── products ── brands
                  |
                  ├── product_images
                  |
                  └── inventory

commission_settings (global, category, or seller scope)
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
| seller_id      | BIGINT        | FK users.id, NOT NULL |
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
| user_id                 | BIGINT        | FK users.id      |
| status                  | VARCHAR(30)   | NOT NULL         |
| payment_method          | VARCHAR(50)   | NOT NULL         |
| payment_status          | VARCHAR(30)   | NOT NULL         |
| subtotal                | NUMERIC(15,2) | NOT NULL         |
| shipping_fee            | NUMERIC(15,2) | NOT NULL DEFAULT 0.00 |
| discount_amount         | NUMERIC(15,2) | NOT NULL DEFAULT 0.00 |
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
| order_id     | BIGINT        | FK orders.id |
| product_id   | BIGINT        | FK products.id |
| product_name | VARCHAR(255)  | NOT NULL    |
| sku          | VARCHAR(100)  | NOT NULL    |
| quantity     | INTEGER       | NOT NULL    |
| unit_price   | NUMERIC(15,2) | NOT NULL    |
| subtotal     | NUMERIC(15,2) | NOT NULL    |

Historical product information is copied into the order item.

---

## 17. payments

Payment transaction records for orders.

| Column                | Type          | Constraints          |
| --------------------- | ------------- | -------------------- |
| id                    | BIGSERIAL     | PK                   |
| order_id              | BIGINT        | FK orders.id, NOT NULL |
| payment_method        | VARCHAR(50)   | NOT NULL             |
| transaction_reference | VARCHAR(255)  | NULL                 |
| amount                | NUMERIC(15,2) | NOT NULL             |
| status                | VARCHAR(30)   | NOT NULL             |
| created_at            | TIMESTAMP     | NOT NULL             |
| updated_at            | TIMESTAMP     | NOT NULL             |

Statuses: `PENDING`, `PAID`, `FAILED`, `REFUNDED`.

---

## 18. reviews

Customer reviews and ratings for purchased products.

| Column               | Type        | Constraints          |
| -------------------- | ----------- | -------------------- |
| id                   | BIGSERIAL   | PK                   |
| user_id              | BIGINT      | FK users.id, NOT NULL |
| product_id           | BIGINT      | FK products.id, NOT NULL |
| order_item_id        | BIGINT      | FK order_items.id, NULL |
| rating               | INTEGER     | NOT NULL, CHECK (rating BETWEEN 1 AND 5) |
| comment              | TEXT        | NULL                 |
| is_verified_purchase | BOOLEAN     | NOT NULL DEFAULT TRUE |
| created_at           | TIMESTAMP   | NOT NULL             |
| updated_at           | TIMESTAMP   | NOT NULL             |

Unique constraint: `(user_id, product_id, order_item_id)` prevents duplicate reviews for the same order purchase.

---

## 19. conversations

AI conversations.

| Column     | Type      | Constraints |
| ---------- | --------- | ----------- |
| id         | UUID      | PK          |
| user_id    | BIGINT    | FK users.id |
| created_at | TIMESTAMP | NOT NULL    |
| updated_at | TIMESTAMP | NOT NULL    |

---

## 20. conversation_messages

Messages inside AI conversations.

| Column          | Type        | Constraints |
| --------------- | ----------- | ----------- |
| id              | BIGSERIAL   | PK          |
| conversation_id | UUID        | FK conversations.id |
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

## 21. coupons

Promotional coupons and vouchers.

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | BIGSERIAL | PK | Primary identifier |
| code | VARCHAR(50) | UNIQUE, NOT NULL | Alphanumeric coupon code (uppercase) |
| name | VARCHAR(150) | NOT NULL | Display name of the campaign |
| description | TEXT | NULL | Promotional details and terms |
| discount_type | VARCHAR(30) | NOT NULL | `PERCENTAGE`, `FIXED_AMOUNT`, `SHIPPING` |
| discount_value | NUMERIC(15,2) | NOT NULL | Discount percentage or direct monetary deduction |
| max_discount_amount | NUMERIC(15,2) | NULL | Maximum discount cap (mandatory for `PERCENTAGE`) |
| min_order_value | NUMERIC(15,2) | NOT NULL DEFAULT 0.00 | Minimum cart subtotal required |
| usage_limit | INTEGER | NOT NULL | Maximum total platform redemptions |
| used_count | INTEGER | NOT NULL DEFAULT 0 | Counter for active redemptions |
| usage_limit_per_user | INTEGER | NOT NULL DEFAULT 1 | Maximum redemptions per individual customer |
| is_public | BOOLEAN | NOT NULL DEFAULT TRUE | Discoverable in public listings vs private code-only |
| start_date | TIMESTAMP | NOT NULL | Validity start timestamp |
| end_date | TIMESTAMP | NOT NULL | Validity end timestamp |
| status | VARCHAR(30) | NOT NULL DEFAULT 'ACTIVE' | `ACTIVE`, `INACTIVE` |
| created_at | TIMESTAMP | NOT NULL DEFAULT CURRENT_TIMESTAMP | Created timestamp |
| updated_at | TIMESTAMP | NOT NULL DEFAULT CURRENT_TIMESTAMP | Last updated timestamp |

---

## 22. coupon_usages

Audit log of coupon redemptions tied to orders and customer accounts.

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | BIGSERIAL | PK | Primary identifier |
| coupon_id | BIGINT | FK coupons.id, NOT NULL | Applied coupon |
| user_id | BIGINT | FK users.id, NOT NULL | Customer who redeemed |
| order_id | BIGINT | FK orders.id, NOT NULL | Order where coupon was applied |
| discount_amount | NUMERIC(15,2) | NOT NULL | Actual discount amount deducted |
| status | VARCHAR(30) | NOT NULL DEFAULT 'USED' | `USED`, `RELEASED` |
| used_at | TIMESTAMP | NOT NULL DEFAULT CURRENT_TIMESTAMP | Redemption timestamp |
| released_at | TIMESTAMP | NULL | Rollback timestamp on order cancellation |

Unique constraint: `order_id` (single coupon per order).
Index constraint: `(coupon_id, user_id, status)`.

---

## 23. commission_settings

Stores marketplace commission fee configurations across global, category, and merchant scopes.

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | BIGSERIAL | PK | Primary identifier |
| scope | VARCHAR(30) | NOT NULL | Scope level: `GLOBAL`, `CATEGORY`, `SELLER` |
| target_id | BIGINT | NULL | Reference to `categories.id` or `users.id` (NULL for `GLOBAL`) |
| rate | NUMERIC(5,4) | NOT NULL | Commission rate, e.g. `0.0500` for 5% ($0 \le \text{rate} \le 1$) |
| description | VARCHAR(255) | NULL | Administrative note or contractual rationale |
| is_active | BOOLEAN | NOT NULL DEFAULT true | Whether rule is actively evaluated |
| created_at | TIMESTAMP | NOT NULL DEFAULT CURRENT_TIMESTAMP | Created timestamp |
| updated_at | TIMESTAMP | NOT NULL DEFAULT CURRENT_TIMESTAMP | Last updated timestamp |

Unique constraint: `(scope, target_id)`.
Index: `(scope, target_id, is_active)`.

---

## 24. seller_order_settlements

Item-level financial ledger tracking merchant earnings, platform commissions, and escrow payout status.

| Column | Type | Constraints | Description |
|---|---|---|---|
| id | BIGSERIAL | PK | Primary identifier |
| order_id | BIGINT | FK orders.id, NOT NULL | Associated customer order |
| order_item_id | BIGINT | FK order_items.id, NOT NULL | Specific purchased item line |
| seller_id | BIGINT | FK users.id, NOT NULL | Merchant owning the sold product |
| item_subtotal | NUMERIC(15,2) | NOT NULL | Item gross sales value (`unit_price * quantity`) |
| commission_rate | NUMERIC(5,4) | NOT NULL | Snapshot commission rate applied at order time |
| commission_amount | NUMERIC(15,2) | NOT NULL | Fee retained by marketplace (`item_subtotal * commission_rate`) |
| platform_discount_subsidized | NUMERIC(15,2) | NOT NULL DEFAULT 0.00 | Platform voucher subsidy compensated to seller |
| seller_net_amount | NUMERIC(15,2) | NOT NULL | Net merchant earnings (`item_subtotal - commission_amount + subsidy`) |
| status | VARCHAR(30) | NOT NULL DEFAULT 'PENDING' | Escrow lifecycle: `PENDING`, `SETTLED`, `CANCELLED` |
| settled_at | TIMESTAMP | NULL | Timestamp when escrow was released to seller balance |
| created_at | TIMESTAMP | NOT NULL DEFAULT CURRENT_TIMESTAMP | Created timestamp |
| updated_at | TIMESTAMP | NOT NULL DEFAULT CURRENT_TIMESTAMP | Last updated timestamp |

Indexes:
- `(seller_id, created_at)`
- `(seller_id, status)`
- `(order_id)`

---

## 25. Indexes

Initial indexes:

```text
users(email)

products(seller_id)
products(sku)
products(slug)
products(category_id)
products(brand_id)
products(status)

inventory(product_id)

orders(order_number)
orders(user_id)
orders(status)
orders(payment_status)

order_items(order_id)
order_items(product_id)

payments(order_id)
payments(transaction_reference)

reviews(product_id)
reviews(user_id)

coupons(code)
coupons(is_public, status, start_date, end_date)

coupon_usages(coupon_id, user_id, status)
coupon_usages(order_id)

commission_settings(scope, target_id, is_active)

seller_order_settlements(seller_id, created_at)
seller_order_settlements(seller_id, status)
seller_order_settlements(order_id)

cart_items(cart_id)

addresses(user_id)

conversation_messages(conversation_id)
```

Additional indexes should be introduced based on actual query patterns.

---

## 26. Database Principles

The database follows these principles:

1. PostgreSQL is the source of truth for transactional data.
2. Foreign keys should protect relational integrity.
3. Prices, commission rates, and payouts in orders are historical snapshots.
4. Product specifications use JSONB for flexibility.
5. Soft deletion should be preferred for important business entities.
6. Database migrations must be version controlled.
7. Application code must not manually modify production schemas.
8. Schema changes must go through migration files.

