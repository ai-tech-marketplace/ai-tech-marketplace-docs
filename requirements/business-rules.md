# Business Rules

## 1. User Rules

### BR-USER-001 — Unique Email

Each user must have a unique email address.

### BR-USER-002 — User Status

A user can have one of the following statuses:

```text
ACTIVE
INACTIVE
SUSPENDED
```

Inactive or suspended users cannot perform protected customer operations.

## 2. Product Rules

### BR-PRODUCT-001 — Product Name

A product must have a non-empty name.

### BR-PRODUCT-002 — Product Price

Product price must be greater than or equal to zero.

### BR-PRODUCT-003 — Product Status

A product can have:

```text
ACTIVE
INACTIVE
OUT_OF_STOCK
```

### BR-PRODUCT-004 — SKU

Each product SKU must be unique.

## 3. Inventory Rules

### BR-INVENTORY-001 — Available Quantity

Available quantity is calculated as:

```text
available_quantity =
    total_quantity - reserved_quantity
```

### BR-INVENTORY-002 — Negative Inventory

Available inventory must never become negative.

### BR-INVENTORY-003 — Reservation

Inventory must be validated before an order is confirmed.

### BR-INVENTORY-004 — Reservation Release

Reserved inventory must be released when an order is cancelled before fulfillment.

## 4. Cart Rules

### BR-CART-001 — Positive Quantity

Cart item quantity must be greater than zero.

### BR-CART-002 — Product Availability

A customer cannot add an unavailable product to the cart.

### BR-CART-003 — Cart Ownership

A customer can only access their own cart.

### BR-CART-004 — Price and Availability Synchronization

When loading an existing cart, product prices and active statuses are checked against current database values. If a product price changed or a product became unavailable or out-of-stock, the response must reflect current values and inform the customer accordingly before checkout.

### BR-CART-005 — Guest Cart Merge

When an unauthenticated guest customer authenticates (login or register), any active guest items stored on the client should be merged into their persistent account cart via a merge request. If a product already exists in the cart, quantities are combined without exceeding available inventory.

## 5. Order Rules

### BR-ORDER-001 — Order Status

Orders maintain two distinct lifecycle dimensions:

1. **Fulfillment Status:**
```text
PENDING
CONFIRMED
PROCESSING
SHIPPED
DELIVERED
CANCELLED
```

2. **Payment Status:**
```text
PENDING
PAID
FAILED
REFUNDED
```

### BR-ORDER-002 — Order State Transition

Valid fulfillment transitions:

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

Cancellation transitions:

```text
PENDING   ───> CANCELLED
CONFIRMED ───> CANCELLED
```

### BR-ORDER-003 — Order Price Snapshot

The order item must store the product price at the time of order creation.

Example:

```text
Product current price: 25,000,000

Order item price: 23,000,000
```

If the product price later changes, the existing order must remain unchanged.

### BR-ORDER-004 — Order Ownership

A customer can only view their own orders. Administrative staff or product sellers can view orders per their granted permissions.

### BR-ORDER-005 — Order Expiration and Auto-Cancellation

Orders placed with online payment methods (e.g. VNPAY, MoMo, Card) that remain in `PENDING` payment status past an expiration window (default 15 minutes) are automatically transitioned to `CANCELLED` by the system, and their reserved inventory is immediately released.

### BR-ORDER-006 — Customer Cancellation Window

A customer may cancel an order directly while it is in `PENDING` or `CONFIRMED` status. Once an order transitions to `PROCESSING` (seller packaging) or `SHIPPED`, customer self-cancellation is disallowed and requires support / return handling.

## 6. Checkout Rules

### BR-CHECKOUT-001 — Inventory Validation

The system must validate inventory before creating a confirmed order.

### BR-CHECKOUT-002 — Empty Cart

A customer cannot checkout with an empty cart.

### BR-CHECKOUT-003 — Order Total

The order total must be calculated from order item snapshots:

```text
subtotal = sum(item.quantity * item.unit_price)
```

The client must not be trusted to provide the final order total.

### BR-CHECKOUT-004 — Total Calculation Breakdown

The final payable amount is computed as:

```text
final_total = subtotal + shipping_fee - discount_amount
```

Where:
- `subtotal` is computed server-side from product price snapshots.
- `shipping_fee` is determined based on the destination shipping address and shipping tier.
- `discount_amount` is validated against active promotion or coupon rules.


## 7. Authorization Rules

### BR-AUTH-001 — Role-Based Access

The system uses role-based authorization.

Initial roles:

```text
CUSTOMER
SELLER
ADMIN
```

### BR-AUTH-002 — Admin Operations

Administrative operations require the `ADMIN` role.

### BR-AUTH-003 — Seller Operations

Seller product management operations require the `SELLER` role or appropriate administrative privileges.

## 8. AI Rules

### BR-AI-001 — Product Grounding

Product recommendations should be based on product information available from the marketplace data source.

### BR-AI-002 — No Fabricated Product Specifications

The AI Assistant must not invent product specifications.

If information is unavailable, the response should clearly indicate that the information is unavailable.

### BR-AI-003 — Product Availability

AI recommendations should consider product availability where possible.

### BR-AI-004 — Price Information

When presenting product prices, the AI service should use current product data from the backend.

### BR-AI-005 — AI Failure

If the AI service is unavailable, the core marketplace should continue to operate without AI functionality.

## 9. Data Consistency Rules

### BR-DATA-001 — Transactional Order Creation

Order creation and related inventory operations must maintain transactional consistency.

### BR-DATA-002 — Server-Side Validation

Important business rules must be validated by the backend.

The frontend must not be considered a trusted source.

### BR-DATA-003 — Auditability

Important state-changing operations should be logged or auditable where appropriate.

## 10. Payment Rules

### BR-PAYMENT-001 — Payment Methods

The system supports the following initial payment methods:

```text
COD (Cash on Delivery)
VNPAY (Online Payment Gateway)
MOMO (E-Wallet)
BANK_TRANSFER (Manual or Automated Transfer)
```

### BR-PAYMENT-002 — Gateway Callback Verification

Online payment transactions must be verified via cryptographic signature validation (HMAC SHA-512 / RSA) upon receipt of webhook or IPN callbacks before marking the payment as `PAID` and the order as `CONFIRMED`.

### BR-PAYMENT-003 — Refund on Cancellation

If an order with `PAID` status is cancelled prior to shipping, the system must trigger an automated or administrative refund process. The payment status transitions to `REFUNDED`.

## 11. Review and Rating Rules

### BR-REVIEW-001 — Verified Purchase Requirement

A customer may only submit a product review if they have purchased that product in an order whose status is `DELIVERED`.

### BR-REVIEW-002 — Rating Scale

Product ratings must be an integer between 1 and 5 stars.

### BR-REVIEW-003 — Review Uniqueness

A customer can submit at most one active review per purchased product per order item.

## 12. Seller and Multi-Vendor Rules

### BR-SELLER-001 — Product Ownership

Each product is associated with a selling entity (`seller_id`). Only the owner seller or an administrator can edit, update inventory, or deactivate the product.

### BR-SELLER-002 — Order Item Fulfillment Visibility

Sellers can view and manage fulfillment only for order items that belong to products they own.

## 13. Promotion & Coupon Rules

### BR-COUPON-001 — Single Coupon per Order

A customer order may apply at most one coupon code. Coupon stacking across items or combining multiple promotional codes on a single order is strictly disallowed.

### BR-COUPON-002 — Discount Types & Calculation Formulas

The system supports three distinct promotional discount types:

1. **`PERCENTAGE`**: Computes a percentage reduction on the order merchandise subtotal, capped at an enforced maximum amount:
   ```text
   discount_amount = min(subtotal * (discount_value / 100), max_discount_amount)
   ```
2. **`FIXED_AMOUNT`**: Deducts a flat currency amount directly from the merchandise subtotal:
   ```text
   discount_amount = min(subtotal, discount_value)
   ```
3. **`SHIPPING`**: Deducts an amount from the calculated delivery fee, capped at the actual shipping fee:
   ```text
   discount_amount = min(shipping_fee, discount_value)
   ```

### BR-COUPON-003 — Minimum Order Threshold

A coupon code cannot be applied if the merchandise subtotal is below its required threshold:
```text
subtotal < min_order_value
```

### BR-COUPON-004 — Usage & Budget Limits

The platform enforces two levels of usage limits:

1. **Global Limit:** A coupon is invalid if its total redemption count across all customers reaches or exceeds its overall quota:
   ```text
   used_count >= usage_limit
   ```
2. **Per-Customer Limit:** A coupon is invalid for a specific customer if that customer's active redemptions reach the per-user limit (default 1):
   ```text
   user_usage_count >= usage_limit_per_user
   ```

### BR-COUPON-005 — Time Validity and Expiration

A coupon is valid only when:
```text
start_date <= current_timestamp <= end_date AND status = 'ACTIVE'
```
Expired status is evaluated dynamically based on whether `current_timestamp > end_date`.

### BR-COUPON-006 — Order Total Floor

The final payable order amount (`subtotal + shipping_fee - discount_amount`) must never be reduced below 0 VND.

### BR-COUPON-007 — Coupon Quota Release and Rollback

If an order is cancelled or times out unpaid:

1. **Auto-Cancellation on 15-min Timeout (`BR-ORDER-005`):** When an online order expires unpaid, the associated `coupon_usages` record transitions to `status = 'RELEASED'` with `released_at = CURRENT_TIMESTAMP`, and `coupons.used_count` is atomically decremented by 1.
2. **Customer Cancellation (`BR-ORDER-006`):** When a customer cancels a `PENDING` or `CONFIRMED` order prior to shipping, the coupon usage is transitioned to `RELEASED` and quota is restored.
3. If the coupon has passed its `end_date` at the time of order cancellation, the usage quota is released in audit records, but the customer cannot reuse the coupon due to `BR-COUPON-005`.

### BR-COUPON-008 — Admin Configuration Immutability

Once a coupon has been redeemed at least once (`used_count > 0`), its core promotional and financial terms (`code`, `discount_type`, `discount_value`, `max_discount_amount`, `min_order_value`) are immutable to preserve transactional accounting consistency. Administrators may only update metadata (`name`, `description`), extend `end_date`, increase `usage_limit`, or deactivate the coupon (`status = 'INACTIVE'`).

