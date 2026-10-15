# Multi-Tier Financial Analytics API Contract

## 1. Overview

The Analytics APIs provide real-time aggregation and financial reporting across three operational tiers:
1. **Platform Administrators (`/api/v1/admin/analytics`)**: Marketplace GMV, net platform commission earnings, merchant leaderboard, and category distribution.
2. **Merchants / Sellers (`/api/v1/seller/analytics`)**: Gross sales, commission deductions, net payout earnings, pending escrow settlements, and top-selling SKUs.
3. **Customers (`/api/v1/customer/analytics`)**: Total personal expenditure, cumulative promotional savings, and category spending distributions.

---

## 2. Common Query Parameters

Where applicable, analytical endpoints accept:
- `period`: Shortcut string (`7d`, `30d`, `this_month`, `last_month`).
- `startDate`: ISO 8601 Date (`YYYY-MM-DD` or `YYYY-MM-DDTHH:mm:ssZ`).
- `endDate`: ISO 8601 Date.
- `limit`: Number of top entries to return (default: `10`, max: `50`).

If neither `period` nor `startDate`/`endDate` is provided, endpoints default to the last 30 calendar days.

---

## 3. Platform Administration Analytics (`/api/v1/admin/analytics`)

Access Requirement: `ROLE_ADMIN`

### 3.1 Platform Financial Overview

Retrieve high-level marketplace financial metrics.

```http
GET /api/v1/admin/analytics/overview?period=30d
```

Response (`200 OK`):

```json
{
  "success": true,
  "data": {
    "currency": "VND",
    "startDate": "2026-09-15T00:00:00Z",
    "endDate": "2026-10-15T23:59:59Z",
    "gmv": 1585000000.00,
    "netPlatformRevenue": 63400000.00,
    "totalOrders": 128,
    "completedOrders": 115,
    "cancelledOrders": 9,
    "cancellationRate": 7.03,
    "totalDiscountsSubsidized": 12500000.00,
    "averageOrderValue": 12382812.50
  },
  "error": null,
  "timestamp": "2026-10-15T15:00:00Z"
}
```

---

### 3.2 Top Performing Sellers

Leaderboard of merchants sorted by generated GMV or net commission.

```http
GET /api/v1/admin/analytics/top-sellers?limit=5&period=30d
```

Response (`200 OK`):

```json
{
  "success": true,
  "data": [
    {
      "sellerId": 12,
      "sellerName": "Asus Official Flagship",
      "totalOrders": 42,
      "grossSales": 680000000.00,
      "commissionPaid": 17000000.00,
      "sellerNetAmount": 663000000.00
    },
    {
      "sellerId": 8,
      "sellerName": "TechZone Gear",
      "totalOrders": 35,
      "grossSales": 420000000.00,
      "commissionPaid": 21000000.00,
      "sellerNetAmount": 399000000.00
    }
  ],
  "error": null,
  "timestamp": "2026-10-15T15:00:00Z"
}
```

---

### 3.3 Category GMV Breakdown

Revenue contribution segmented by product category.

```http
GET /api/v1/admin/analytics/top-categories?period=30d
```

Response (`200 OK`):

```json
{
  "success": true,
  "data": [
    {
      "categoryId": 1,
      "categoryName": "Laptops & Notebooks",
      "totalUnitsSold": 45,
      "gmv": 950000000.00,
      "percentageShare": 59.94,
      "platformCommissionEarned": 23750000.00
    },
    {
      "categoryId": 2,
      "categoryName": "PC Components & GPUs",
      "totalUnitsSold": 38,
      "gmv": 410000000.00,
      "percentageShare": 25.87,
      "platformCommissionEarned": 14350000.00
    },
    {
      "categoryId": 5,
      "categoryName": "Accessories & Peripherals",
      "totalUnitsSold": 120,
      "gmv": 225000000.00,
      "percentageShare": 14.19,
      "platformCommissionEarned": 18000000.00
    }
  ],
  "error": null,
  "timestamp": "2026-10-15T15:00:00Z"
}
```

---

### 3.4 Platform Daily / Monthly Revenue Trajectory

Time-series data for administrative revenue charting.

```http
GET /api/v1/admin/analytics/revenue-chart?period=7d&interval=daily
```

Response (`200 OK`):

```json
{
  "success": true,
  "data": {
    "interval": "daily",
    "timeline": [
      {
        "date": "2026-10-09",
        "gmv": 180000000.00,
        "commission": 7200000.00,
        "orderCount": 16
      },
      {
        "date": "2026-10-10",
        "gmv": 240000000.00,
        "commission": 9600000.00,
        "orderCount": 21
      }
    ]
  },
  "error": null,
  "timestamp": "2026-10-15T15:00:00Z"
}
```

---

## 4. Merchant / Seller Analytics (`/api/v1/seller/analytics`)

Access Requirement: `ROLE_SELLER` (Scoped automatically to authenticated merchant)

### 4.1 Seller Store Overview

Store performance, gross revenue, commission deducted, and payout balances.

```http
GET /api/v1/seller/analytics/overview?period=30d
```

Headers:
- `Authorization`: `Bearer <seller_jwt_token>`

Response (`200 OK`):

```json
{
  "success": true,
  "data": {
    "currency": "VND",
    "startDate": "2026-09-15T00:00:00Z",
    "endDate": "2026-10-15T23:59:59Z",
    "grossSales": 420000000.00,
    "commissionPaid": 21000000.00,
    "netEarnings": 399000000.00,
    "pendingSettlement": 45000000.00,
    "availableBalance": 354000000.00,
    "totalOrders": 35,
    "totalUnitsSold": 52
  },
  "error": null,
  "timestamp": "2026-10-15T15:00:00Z"
}
```

---

### 4.2 Top Performing Products for Seller

Best-selling products belonging to the authenticated merchant.

```http
GET /api/v1/seller/analytics/top-products?limit=5&period=30d
```

Response (`200 OK`):

```json
{
  "success": true,
  "data": [
    {
      "productId": 101,
      "productName": "ASUS ROG Zephyrus G16 (2026)",
      "sku": "ROG-G16-2026-01",
      "unitsSold": 14,
      "grossRevenue": 490000000.00,
      "netRevenue": 477750000.00
    },
    {
      "productId": 104,
      "productName": "ROG Harpe Ace Wireless Gaming Mouse",
      "sku": "ROG-HARPE-ACE-BLK",
      "unitsSold": 28,
      "grossRevenue": 84000000.00,
      "netRevenue": 77280000.00
    }
  ],
  "error": null,
  "timestamp": "2026-10-15T15:00:00Z"
}
```

---

### 4.3 Seller Settlement History Ledger

Paginated breakdown of item-level settlements and escrow status.

```http
GET /api/v1/seller/analytics/order-settlements?page=0&size=20&status=SETTLED
```

Response (`200 OK`):

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "settlementId": 501,
        "orderNumber": "ORD-20261014-0012",
        "productName": "ASUS ROG Zephyrus G16 (2026)",
        "quantity": 1,
        "itemSubtotal": 35000000.00,
        "commissionRate": 0.0250,
        "commissionAmount": 875000.00,
        "platformDiscountSubsidized": 0.00,
        "sellerNetAmount": 34125000.00,
        "status": "SETTLED",
        "settledAt": "2026-10-14T14:30:00Z",
        "createdAt": "2026-10-07T10:15:00Z"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 1,
    "totalPages": 1
  },
  "error": null,
  "timestamp": "2026-10-15T15:00:00Z"
}
```

---

## 5. Customer Personal Spending Analytics (`/api/v1/customer/analytics`)

Access Requirement: `ROLE_CUSTOMER` (Scoped automatically to authenticated user)

### 5.1 Personal Spending Summary

Overview of total technology purchases, order counts, and promotional savings.

```http
GET /api/v1/customer/analytics/spending-summary
```

Headers:
- `Authorization`: `Bearer <customer_jwt_token>`

Response (`200 OK`):

```json
{
  "success": true,
  "data": {
    "currency": "VND",
    "totalSpent": 48500000.00,
    "totalSaved": 2800000.00,
    "totalOrdersPlaced": 6,
    "completedOrders": 5,
    "firstOrderDate": "2026-08-01T10:00:00Z",
    "lastOrderDate": "2026-10-12T16:20:00Z"
  },
  "error": null,
  "timestamp": "2026-10-15T15:00:00Z"
}
```

---

### 5.2 Category Spending Breakdown

Categorical distribution of the customer's hardware and tech investments.

```http
GET /api/v1/customer/analytics/category-breakdown
```

Response (`200 OK`):

```json
{
  "success": true,
  "data": [
    {
      "categoryId": 1,
      "categoryName": "Laptops & Notebooks",
      "amountSpent": 35000000.00,
      "percentageShare": 72.16,
      "itemsPurchased": 1
    },
    {
      "categoryId": 5,
      "categoryName": "Accessories & Peripherals",
      "amountSpent": 13500000.00,
      "percentageShare": 27.84,
      "itemsPurchased": 4
    }
  ],
  "error": null,
  "timestamp": "2026-10-15T15:00:00Z"
}
```

---

### 5.3 Monthly Spending Trajectory

Monthly historical spend curve over the preceding 6 or 12 months.

```http
GET /api/v1/customer/analytics/monthly-spending?months=6
```

Response (`200 OK`):

```json
{
  "success": true,
  "data": {
    "months": 6,
    "timeline": [
      {
        "month": "2026-05",
        "amountSpent": 0.00,
        "orderCount": 0
      },
      {
        "month": "2026-08",
        "amountSpent": 35000000.00,
        "orderCount": 1
      },
      {
        "month": "2026-09",
        "amountSpent": 5500000.00,
        "orderCount": 2
      },
      {
        "month": "2026-10",
        "amountSpent": 8000000.00,
        "orderCount": 2
      }
    ]
  },
  "error": null,
  "timestamp": "2026-10-15T15:00:00Z"
}
```
