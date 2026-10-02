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

## 5. Order Rules

### BR-ORDER-001 — Order Status

Orders can have the following states:

```text
PENDING
CONFIRMED
PROCESSING
SHIPPED
DELIVERED
CANCELLED
```

### BR-ORDER-002 — Order State Transition

Valid transitions:

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

Cancellation is allowed only from states where business rules permit cancellation.

### BR-ORDER-003 — Order Price Snapshot

The order item must store the product price at the time of order creation.

Example:

```text
Product current price: 25,000,000

Order item price: 23,000,000
```

If the product price later changes, the existing order must remain unchanged.

### BR-ORDER-004 — Order Ownership

A customer can only view their own orders.

## 6. Checkout Rules

### BR-CHECKOUT-001 — Inventory Validation

The system must validate inventory before creating a confirmed order.

### BR-CHECKOUT-002 — Empty Cart

A customer cannot checkout with an empty cart.

### BR-CHECKOUT-003 — Order Total

The order total must be calculated from order item snapshots.

```text
order_total =
    sum(item.quantity * item.unit_price)
```

The client must not be trusted to provide the final order total.

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
