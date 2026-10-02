# Order API Contract

## 1. Overview

Order APIs handle checkout and order management.

Base URL:

```text
/api/v1/orders
```

All customer endpoints require authentication.

---

# 2. Create Order

```http
POST /api/v1/orders
```

Request:

```json
{
  "addressId": 10
}
```

The backend obtains the customer's current cart.

The client does not provide:

* Total amount
* Unit prices
* Product names
* Inventory information

The backend calculates these values.

---

# 3. Order Creation Flow

```text
POST /orders
      |
      v
Validate User
      |
      v
Load Cart
      |
      v
Load Products
      |
      v
Validate Product Status
      |
      v
Validate Inventory
      |
      v
Calculate Total
      |
      v
Create Order
      |
      v
Create Order Items
      |
      v
Reserve Inventory
      |
      v
Publish OrderCreated
      |
      v
Return Order
```

---

# 4. Create Order Response

```json
{
  "success": true,
  "data": {
    "id": 100,
    "orderNumber": "ORD-20260001",
    "status": "PENDING",
    "totalAmount": 59980000,
    "items": [
      {
        "productId": 1,
        "productName": "Gaming Laptop",
        "sku": "LAP-001",
        "quantity": 2,
        "unitPrice": 29990000,
        "subtotal": 59980000
      }
    ],
    "shippingAddress": {
      "recipientName": "John Doe",
      "phone": "0900000000",
      "addressLine": "123 Example Street",
      "district": "Cau Giay",
      "city": "Hanoi"
    }
  },
  "message": "Order created successfully"
}
```

---

# 5. Get User Orders

```http
GET /api/v1/orders
```

Query:

```text
page
size
status
```

Example:

```http
GET /api/v1/orders?page=0&size=20&status=DELIVERED
```

---

# 6. Get Order Detail

```http
GET /api/v1/orders/{id}
```

The backend must verify that the current user owns the order unless the caller has an administrative role.

---

# 7. Cancel Order

```http
POST /api/v1/orders/{id}/cancel
```

The backend determines whether cancellation is allowed.

The frontend must not determine order state transitions.

---

# 8. Order Status

Initial statuses:

```text
PENDING
CONFIRMED
PROCESSING
SHIPPED
DELIVERED
CANCELLED
```

Expected lifecycle:

```text
PENDING
   |
   v
CONFIRMED
   |
   v
PROCESSING
   |
   v
SHIPPED
   |
   v
DELIVERED
```

Cancellation:

```text
PENDING ──────> CANCELLED
CONFIRMED ────> CANCELLED
```

Actual allowed transitions are enforced by the backend.

---

# 9. Order Events

After important state changes, the backend may publish Kafka events.

Example:

```text
OrderCreated
OrderConfirmed
OrderCancelled
OrderShipped
OrderDelivered
```

Example:

```text
Order Service
     |
     | OrderCreated
     v
   Kafka
     |
     ├── Inventory Consumer
     ├── Notification Consumer
     └── Analytics Consumer
```

---

# 10. Inventory Consistency

Inventory operations must be handled transactionally.

The system must prevent:

```text
available inventory = 5

Customer A buys 4
Customer B buys 4

Result:
8 sold
5 available
```

The backend must use appropriate concurrency control.

Redis distributed locking may be introduced where required, but PostgreSQL remains the transactional source of truth.
