# Seed Data

## 1. Purpose

Seed data provides a consistent development environment.

Seed data is not production data.

---

## 2. Roles

Initial roles:

```text
CUSTOMER
SELLER
ADMIN
```

---

## 3. Categories

Initial categories:

```text
Laptop
Smartphone
Tablet
Monitor
Keyboard
Mouse
Headphone
PC Component
```

---

## 4. Brands

Initial brands:

```text
ASUS
Lenovo
Dell
HP
Apple
Samsung
Logitech
Sony
```

---

## 5. Development Admin

A development administrator may be created for local testing.

Example:

```text
Email:
admin@example.com
```

The password must be provided through environment configuration and must not be committed to Git.

---

## 6. Example Products

Development data should contain enough products to test:

* Product listing
* Search
* Filtering
* Pagination
* Product detail
* Cart
* Checkout
* AI recommendation

Example product categories:

```text
5 laptops
5 smartphones
3 monitors
3 keyboards
3 mice
3 headphones
```

---

## 7. AI Test Products

Some products should contain detailed specifications so that AI functionality can be tested.

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

Another example:

```json
{
  "cpu": "AMD Ryzen 7",
  "gpu": "RTX 4060",
  "ram": "16GB",
  "storage": "1TB SSD",
  "display": "15.6 inch IPS"
}
```

---

## 8. Seed Data Rules

Seed data must:

* Be deterministic
* Be reproducible
* Not contain production information
* Not contain real customer data
* Not contain real passwords
* Be safe to execute repeatedly or detect existing records

---

## 9. AI Development Dataset

The AI service may later use a generated product dataset for:

* Embeddings
* Semantic search
* Recommendation
* RAG testing

This dataset should remain separate from production data.
