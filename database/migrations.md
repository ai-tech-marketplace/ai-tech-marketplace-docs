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

## 5. Initial Migration Plan

### V1

Create:

```text
users
```

### V2

Create:

```text
roles
user_roles
```

### V3

Create:

```text
categories
brands
products
product_images
```

### V4

Create:

```text
inventory
```

### V5

Create:

```text
addresses
carts
cart_items
```

### V6

Create:

```text
orders
order_items
```

### V7

Create:

```text
conversations
conversation_messages
```

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
