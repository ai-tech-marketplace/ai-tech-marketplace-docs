# Database Migrations

## 1. Migration Tool

The Spring Boot backend will use:

```text
Flyway
```

for database schema versioning.

---

## 2. Migration Location

Migration files will live inside the backend repository:

```text
src/main/resources/db/migration/
```

Example:

```text
V1__create_users.sql
V2__create_roles.sql
V3__create_products.sql
V4__create_inventory.sql
V5__create_carts.sql
V6__create_orders.sql
V7__create_ai_conversations.sql
```

---

## 3. Migration Rules

Migration files must:

* Never be modified after being applied
* Have unique version numbers
* Be committed to Git
* Be reviewed before merging
* Be executed automatically by the application in development

---

## 4. Version Naming

Format:

```text
V<version>__<description>.sql
```

Example:

```text
V1__create_users.sql
```

```text
V2__create_roles.sql
```

```text
V3__create_categories.sql
```

---

## 5. Migration Sequence Plan

### Applied Migrations (Core Baseline):

* **V1__init.sql**: Base `users`, `roles`, and `user_roles` tables.
* **V2__create_products.sql**: Core `products` table with basic seller link and pricing checks.
* **V3__create_categories.sql**: `categories` table.
* **V4__create_product_categories.sql**: `product_categories` join table.
* **V5__create_product_images.sql**: `product_images` object storage mapping.
* **V6__create_carts.sql**: `carts` and `cart_items` tables with ownership and quantity constraints.

### Upcoming Migrations:

* **V7__create_brands.sql**: `brands` table and `brand_id` reference on `products`.
* **V8__create_addresses.sql**: `addresses` table for customer shipping book.
* **V9__create_inventory.sql**: Dedicated `inventory` table with `quantity` and `reserved_quantity`.
* **V10__create_orders.sql**: `orders`, `order_items`, and `payments` tables with payment states and fee columns.
* **V11__create_reviews.sql**: `reviews` table with 1-5 star ratings and verified purchase constraint.
* **V12__create_ai_conversations.sql**: `conversations` and `conversation_messages` tables.
* **V13__create_coupons.sql**: `coupons` table (with `is_public`) and `coupon_usages` table (with `status` and `released_at`).

---

## 6. Development Database

Development PostgreSQL will run using Docker Compose.

Example:

```text
PostgreSQL
localhost:5432
```

Database:

```text
ai_marketplace
```

---

## 7. Production Database

Production database configuration must be provided through environment variables.

The application must not contain production credentials in Git.

---

## 8. Migration Flow

```text
Developer
    |
    v
Create migration
    |
    v
Git commit
    |
    v
Pull Request
    |
    v
Review
    |
    v
Merge
    |
    v
CI/CD
    |
    v
Deploy
    |
    v
Flyway migration
```

---

## 9. Rollback

Flyway migrations should generally use forward migrations rather than modifying old migration files.

Example:

Incorrect:

```text
Modify V5
```

Correct:

```text
Create V8__fix_cart_constraint.sql
```

This preserves migration history.
