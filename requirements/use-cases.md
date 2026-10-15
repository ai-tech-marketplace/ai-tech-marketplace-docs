# Use Cases

## 1. Actors

The system contains the following actors:

### Customer

A customer uses the platform to discover and purchase technology products.

### Seller

A seller manages products and inventory.

### Administrator

An administrator manages the platform and monitors the system.

### AI Shopping Assistant

The AI Shopping Assistant helps customers discover and compare products.

### External Services

External services may include:

* LLM provider
* Payment provider
* Email provider
* Object storage provider

## 2. Customer Use Cases

### UC-C01 — Register

**Actor:** Customer

**Goal:** Create a new customer account.

**Flow:**

```text
Customer
   |
   v
Submit registration form
   |
   v
Validate information
   |
   v
Create account
   |
   v
Return success
```

### UC-C02 — Login

**Actor:** Customer

**Goal:** Authenticate into the system.

**Flow:**

```text
Customer
   |
   v
Submit credentials
   |
   v
Validate credentials
   |
   v
Generate JWT
   |
   v
Return authentication response
```

### UC-C03 — Browse Products

**Actor:** Customer

**Goal:** View available products.

The customer can:

* Browse products
* Filter products
* Sort products
* Paginate results

### UC-C04 — View Product

**Actor:** Customer

**Goal:** View detailed product information.

The product detail may include:

* Product name
* Brand
* Category
* Price
* Specifications
* Images
* Availability

### UC-C05 — Add Product to Cart

**Actor:** Customer

**Goal:** Add a product to the shopping cart.

**Flow:**

```text
Customer
   |
   v
Select product
   |
   v
Select quantity
   |
   v
Validate availability
   |
   v
Add item to cart
```

### UC-C06 — Checkout

**Actor:** Customer

**Goal:** Create an order from the shopping cart.

**Flow:**

```text
Customer
   |
   v
Review cart
   |
   v
Select address
   |
   v
Validate inventory
   |
   v
Calculate total
   |
   v
Create order
   |
   v
Reserve inventory
```

### UC-C07 — View Orders

**Actor:** Customer

**Goal:** View previous orders and their statuses.

### UC-C08 — Apply Coupon

**Actor:** Customer

**Goal:** Apply a promotional coupon to reduce cart subtotal or shipping fee before completing order.

**Flow:**

```text
Customer
   |
   v
Enter coupon code (or select public coupon)
   |
   v
Submit coupon validation request (with optional address)
   |
   v
Backend validates rules (dates, min order, usage limits)
   |
   v
Calculate discount breakdown (PERCENTAGE, FIXED_AMOUNT, or SHIPPING)
   |
   v
Display updated order total and discount deduction
```

### UC-C09 — View Personal Spending Analytics

**Actor:** Customer

**Goal:** Inspect personal technology purchase history, total amount spent, savings gained from discounts/coupons, and category distribution.

**Flow:**

```text
Customer
   |
   v
Request spending summary (/api/v1/customer/analytics/spending-summary)
   |
   v
Backend aggregates customer order history
   |
   v
Display total spent, promotional savings, and monthly purchase chart
```

## 3. Seller Use Cases

### UC-S01 — Create Product

Seller creates a new product.

### UC-S02 — Update Product

Seller updates product information.

### UC-S03 — Manage Inventory

Seller updates inventory quantities.

### UC-S04 — View Product Orders

Seller views orders containing their products.

### UC-S05 — View Store Performance & Earnings

**Actor:** Seller

**Goal:** Track store gross sales, commission deductions paid to the platform, net earnings, pending escrow settlements, and top-selling products.

**Flow:**

```text
Seller
   |
   v
Open Seller Analytics Dashboard (/api/v1/seller/analytics/overview)
   |
   v
Backend aggregates store sales and item settlements
   |
   v
Display gross revenue, commission fees, net payout, and top-selling SKUs
```

## 4. Administrator Use Cases

### UC-A01 — Manage Users

Administrator can view, activate, deactivate, and manage users.

### UC-A02 — Manage Products

Administrator can manage product information.

### UC-A03 — Manage Categories

Administrator can create, update, and deactivate categories.

### UC-A04 — Manage Brands

Administrator can manage product brands.

### UC-A05 — Manage Orders

Administrator can view and manage order statuses.

### UC-A06 — View Platform Financial Analytics

**Actor:** Administrator

**Goal:** Monitor overall marketplace health, Gross Merchandise Value (GMV), net commission earnings, voucher subsidy costs, and merchant leaderboards.

**Flow:**

```text
Administrator
   |
   v
Access Admin Analytics Dashboard (/api/v1/admin/analytics/overview)
   |
   v
Query financial metrics across selected time range (e.g., 30 days)
   |
   v
Display GMV, platform revenue, order cancellation rates, and top categories
```

### UC-A07 — Manage Commission Rates

**Actor:** Administrator

**Goal:** Configure, update, or inspect platform commission rates across global, category-specific, and seller-specific scopes.

**Flow:**

```text
Administrator
   |
   v
Access Commission Settings (/api/v1/admin/commission-settings)
   |
   v
Create or update rate rule (e.g. Set Laptops category to 2.5%, Accessories to 8%)
   |
   v
Backend validates rate range (0%–100%) and uniqueness constraints
   |
   v
Rule becomes immediately active for subsequent checkout transactions
```

### UC-A08 — Manage Coupons

Administrator can create promotional coupons, configure discount types and constraints, toggle coupon active status, and view usage metrics.

## 5. AI Use Cases

### UC-AI01 — Product Recommendation

**Actor:** Customer

Customer describes a need:

```text
I need a laptop around 30 million VND
for Java development, Docker,
and occasional gaming.
```

The AI Assistant:

```text
Understand request
       |
       v
Extract requirements
       |
       v
Search product catalog
       |
       v
Rank candidate products
       |
       v
Generate explanation
       |
       v
Return recommendations
```

### UC-AI02 — Product Comparison

Customer asks:

```text
Compare Product A and Product B.
```

The AI Assistant compares:

* Price
* CPU
* GPU
* RAM
* Storage
* Display
* Other relevant specifications

### UC-AI03 — Natural Language Search

Customer can search using natural language rather than exact product keywords.

Example:

```text
Find a lightweight laptop
with at least 32GB RAM
for programming.
```

### UC-AI04 — Product Question Answering

Customer asks questions about a product.

Example:

```text
Can this laptop run Docker
and multiple Java services?
```

The AI Assistant should answer using available product information and relevant knowledge sources.

## 6. Main Customer Journey

```text
Register / Login
       |
       v
Browse Products
       |
       v
Search / Filter
       |
       v
View Product
       |
       +------------------+
       |                  |
       v                  v
Add to Cart         Ask AI Assistant
       |                  |
       v                  v
Review Cart        Get Recommendation
       |                  |
       +--------+---------+
                |
                v
             Checkout
                |
                v
           Create Order
                |
                v
          Order Processing
                |
                v
             Delivered
```
