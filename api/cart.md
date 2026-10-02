# Cart API Contract

## 1. Overview

Cart APIs allow authenticated customers to manage products before checkout.

Base URL:

```text
/api/v1/cart
```

All endpoints require authentication.

---

# 2. Get Current Cart

```http
GET /api/v1/cart
```

Response:

```json
{
  "success": true,
  "data": {
    "id": 1,
    "items": [
      {
        "id": 10,
        "product": {
          "id": 1,
          "name": "Gaming Laptop",
          "price": 29990000,
          "thumbnailUrl": "/images/laptop.jpg"
        },
        "quantity": 2,
        "subtotal": 59980000
      }
    ],
    "totalItems": 2,
    "totalAmount": 59980000
  },
  "message": null
}
```

---

# 3. Add Product to Cart

```http
POST /api/v1/cart/items
```

Request:

```json
{
  "productId": 1,
  "quantity": 2
}
```

Response:

```json
{
  "success": true,
  "data": {
    "id": 10,
    "productId": 1,
    "quantity": 2
  },
  "message": "Product added to cart"
}
```

---

# 4. Update Cart Item

```http
PUT /api/v1/cart/items/{itemId}
```

Request:

```json
{
  "quantity": 3
}
```

---

# 5. Remove Cart Item

```http
DELETE /api/v1/cart/items/{itemId}
```

Response:

```text
204 No Content
```

---

# 6. Clear Cart

```http
DELETE /api/v1/cart/items
```

Response:

```text
204 No Content
```

---

# 7. Cart Rules

The backend must validate:

* Product exists
* Product is active
* Product is purchasable
* Quantity is greater than zero
* User owns the cart
* Quantity does not exceed allowed inventory

The frontend must not determine whether a product is available.

---

# 8. Cart Price

The frontend may display calculated prices.

However, the backend must recalculate all prices when creating an order.

The client must never be trusted to provide the final order total.

Example:

```text
Frontend:

totalAmount = 100,000,000
        |
        X

Backend:

Product prices
     +
Quantities
     +
Business rules
     |
     v
Final order amount
```

---

# 9. Cart Flow

```text
Customer
   |
   v
Product Page
   |
   | Add to cart
   v
POST /cart/items
   |
   v
Backend
   |
   v
PostgreSQL
```
