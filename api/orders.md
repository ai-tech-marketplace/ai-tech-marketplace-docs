# Order API Contract

## 1. Overview

Order APIs handle checkout and order management.

Base URL:

```text
/api/v1/orders
```

All customer endpoints require authentication.

---

## 2. Create Order

```http
POST /api/v1/orders
```

Request:

```json
{
  "addressId": 10,
  "paymentMethod": "VNPAY",
  "couponCode": "TECH2026"
}
```

The backend obtains the customer's current cart and validates product availability, active stock, shipping fees, and coupon validity.

The client does not provide:

* Subtotal or total amount
* Unit prices
* Product names
* Inventory calculations

The backend calculates all amounts authoritative server-side.

---

## 3. Order Creation Flow

```text
POST /orders
      |
      v
Validate User & Address
      |
      v
Load Active Cart
      |
      v
Validate Product Status & Inventory
      |
      v
Validate Coupon (if couponCode provided)
      |
      v
Calculate Subtotal, Shipping Fee, Discounts
      |
      v
Create Order (status=PENDING, paymentStatus=PENDING)
      |
      v
Create Order Items & Payment Record
      |
      v
Record Coupon Usage (coupon_usages: status=USED) & Increment used_count
      |
      v
Reserve Inventory (reserved_quantity += quantity)
      |
      v
Clear Customer Cart Items
      |
      v
Publish Kafka Event OrderCreated
      |
      v
Return Order Response (with Payment URL if online)
```

---

## 4. Create Order Response

```json
{
  "success": true,
  "data": {
    "id": 100,
    "orderNumber": "ORD-20260001",
    "status": "PENDING",
    "paymentMethod": "VNPAY",
    "paymentStatus": "PENDING",
    "subtotal": 59980000,
    "shippingFee": 30000,
    "discountAmount": 500000,
    "totalAmount": 59510000,
    "paymentUrl": "https://sandbox.vnpayment.vn/paymentv2/vpcpay.html?...",
    "expiresAt": "2026-10-13T13:45:00Z",
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
      "ward": "Dich Vong Hau",
      "district": "Cau Giay",
      "city": "Hanoi"
    }
  },
  "message": "Order created successfully"
}
```

---

## 5. Get User Orders

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

## 6. Get Order Detail

```http
GET /api/v1/orders/{id}
```

The backend must verify that the current user owns the order unless the caller has an administrative role.

---

## 7. Cancel Order

```http
POST /api/v1/orders/{id}/cancel
```

The backend determines whether cancellation is allowed.

The frontend must not determine order state transitions.

---

## 8. Order Status and Payment Lifecycle

### 8.1 Status Dimensions

Orders track two orthogonal states:

1. **Fulfillment Status:** `PENDING`, `CONFIRMED`, `PROCESSING`, `SHIPPED`, `DELIVERED`, `CANCELLED`
2. **Payment Status:** `PENDING`, `PAID`, `FAILED`, `REFUNDED`

### 8.2 Expected Lifecycle

```text
       PENDING (Fulfillment) / PENDING (Payment)
          |
   [Payment Successful]
          v
       CONFIRMED (Fulfillment) / PAID (Payment)
          |
   [Seller Packages Items]
          v
       PROCESSING
          |
   [Shipped by Carrier]
          v
       SHIPPED
          |
   [Customer Received]
          v
       DELIVERED
```

### 8.3 Auto-Expiration (TTL)

For orders awaiting online payment, an asynchronous timer or scheduled task checks orders past their `expiresAt` timestamp (default: 15 minutes). If payment is not completed, the order automatically transitions to `CANCELLED`, payment status becomes `FAILED`, reserved stock is released back into available inventory, and any associated coupon redemption is released (`coupon_usages.status = 'RELEASED'`, `released_at = CURRENT_TIMESTAMP`, `coupons.used_count` decremented by 1) per `BR-COUPON-007`.

### 8.4 Customer Cancellation Window

Customers may initiate cancellation (`POST /api/v1/orders/{id}/cancel`) only while the order is in `PENDING` or `CONFIRMED` fulfillment state. Upon cancellation, reserved stock is released, and associated coupon quota is restored with `coupon_usages.status = 'RELEASED'` and `coupons.used_count` decremented by 1. Once in `PROCESSING` or `SHIPPED`, cancellations must go through return/refund support workflows.

---

## 9. Payment Webhook Callback

Gateway callback endpoint for asynchronous IPN / webhook payment confirmation.

```http
POST /api/v1/payments/webhook/{gateway}
```

Example request (`POST /api/v1/payments/webhook/vnpay`):

```json
{
  "orderNumber": "ORD-20260001",
  "gatewayTransactionId": "VNP14892019",
  "amount": 59510000,
  "responseCode": "00",
  "secureHash": "3b29c9a96e1f0e2..."
}
```

Backend actions:
1. Verify cryptographic HMAC signature using gateway secret key.
2. Verify order amount matches database record.
3. Update `payments` record (`status = PAID`).
4. Update `orders` (`payment_status = PAID`, `status = CONFIRMED`).
5. Publish Kafka domain event `OrderConfirmed`.
6. Return `{"RspCode": "00", "Message": "Confirm Success"}` to gateway.

---

## 10. Order Events

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

## 11. Inventory Consistency

Inventory operations must be handled transactionally.

The system must prevent overselling:

```text
available inventory = 5

Customer A buys 4
Customer B buys 4

Result:
8 sold
5 available (INSUFFICIENT!)
```

The backend must use database-level locking (`SELECT ... FOR UPDATE` or optimistic locking with `@Version`) to guarantee atomic stock reservation:

```sql
UPDATE inventory
SET reserved_quantity = reserved_quantity + :qty,
    updated_at = CURRENT_TIMESTAMP
WHERE product_id = :productId
  AND (quantity - reserved_quantity) >= :qty;
```

If the update affects 0 rows, the transaction rolls back with a `422 Unprocessable Entity` ("Insufficient stock").

