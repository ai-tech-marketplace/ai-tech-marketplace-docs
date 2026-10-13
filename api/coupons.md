# Coupon & Promotion API Contract

## 1. Overview

Coupon APIs govern promotional voucher discovery, pre-checkout cart validation, and administrative campaign management.

Base URL:

```text
/api/v1/coupons
/api/v1/admin/coupons
```

---

## 2. Error Codes

| Error Code | HTTP Status | Description |
| --- | --- | --- |
| `COUPON_NOT_FOUND` | 404 | Coupon code does not exist |
| `COUPON_EXPIRED` | 400 | Current timestamp is outside `start_date` and `end_date` |
| `COUPON_INACTIVE` | 400 | Coupon status is set to `INACTIVE` |
| `COUPON_USAGE_LIMIT_EXCEEDED` | 400 | Total campaign quota has been exhausted (`used_count >= usage_limit`) |
| `COUPON_USER_LIMIT_EXCEEDED` | 400 | Customer has already used this coupon up to `usage_limit_per_user` |
| `COUPON_MIN_ORDER_NOT_MET` | 400 | Cart merchandise subtotal is below `min_order_value` |
| `COUPON_ALREADY_EXISTS` | 409 | Coupon code already exists in system |
| `COUPON_IMMUTABLE_FIELD_CHANGE` | 400 | Cannot alter financial terms on coupon with active redemptions |

---

## 3. Customer Endpoints

### 3.1 Get Public Active Coupons

Retrieve currently active public coupons eligible for customers.

```http
GET /api/v1/coupons
```

Headers:
- `Authorization`: `Bearer <token>` (Optional; when present, filters out coupons the user has already exhausted)

Query Parameters:
- `page`: Page number (default: 0)
- `size`: Page size (default: 20)

Response (`200 OK`):

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "id": 1,
        "code": "TECH2026",
        "name": "Tech Festival 2026",
        "description": "10% off for orders from 20,000,000 VND (capped at 500,000 VND)",
        "discountType": "PERCENTAGE",
        "discountValue": 10.0,
        "maxDiscountAmount": 500000.0,
        "minOrderValue": 20000000.0,
        "isPublic": true,
        "startDate": "2026-10-15T00:00:00Z",
        "endDate": "2026-10-31T23:59:59Z"
      },
      {
        "id": 2,
        "code": "FREESHIP",
        "name": "Free Shipping Vouchers",
        "description": "Free shipping up to 30,000 VND for orders from 5,000,000 VND",
        "discountType": "SHIPPING",
        "discountValue": 30000.0,
        "maxDiscountAmount": 30000.0,
        "minOrderValue": 5000000.0,
        "isPublic": true,
        "startDate": "2026-10-01T00:00:00Z",
        "endDate": "2026-10-31T23:59:59Z"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 2,
    "totalPages": 1
  },
  "message": null
}
```

---

### 3.2 Validate Coupon Against Active Cart

Validate a coupon code against the customer's current active shopping cart before placing an order.

```http
POST /api/v1/coupons/validate
```

Headers:
- `Authorization`: `Bearer <token>` (Required)

Request:

```json
{
  "couponCode": "TECH2026",
  "addressId": 10
}
```

Field Notes:
- `couponCode` (string, required): Uppercase coupon code.
- `addressId` (number, optional): Shipping address ID. If provided, enables precise shipping fee calculation for `SHIPPING` type coupons. If omitted for a `SHIPPING` coupon, the response sets `shippingDiscountPending: true` indicating final discount deduction will resolve at checkout upon address selection.

Response (`200 OK` — Success):

```json
{
  "success": true,
  "data": {
    "code": "TECH2026",
    "discountType": "PERCENTAGE",
    "discountValue": 10.0,
    "discountAmount": 500000.0,
    "subtotal": 59980000.0,
    "shippingFee": 30000.0,
    "estimatedTotal": 59510000.0,
    "valid": true,
    "shippingDiscountPending": false,
    "message": "Coupon applied successfully"
  },
  "message": null
}
```

Response (`400 Bad Request` — Validation Failure):

```json
{
  "success": false,
  "data": null,
  "message": "Order subtotal (15,000,000 VND) does not meet minimum order threshold of 20,000,000 VND",
  "errorCode": "COUPON_MIN_ORDER_NOT_MET"
}
```

---

## 4. Administration Endpoints

Administrative operations require the `ADMIN` role.

### 4.1 List All Coupons (Admin)

```http
GET /api/v1/admin/coupons
```

Query Parameters:
- `page`: Page index (default: 0)
- `size`: Page size (default: 20)
- `status`: Optional filter (`ACTIVE`, `INACTIVE`)
- `search`: Keyword query on code or name

Response (`200 OK`):

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "id": 1,
        "code": "TECH2026",
        "name": "Tech Festival 2026",
        "description": "10% off for orders from 20M VND",
        "discountType": "PERCENTAGE",
        "discountValue": 10.0,
        "maxDiscountAmount": 500000.0,
        "minOrderValue": 20000000.0,
        "usageLimit": 1000,
        "usedCount": 142,
        "usageLimitPerUser": 1,
        "isPublic": true,
        "startDate": "2026-10-15T00:00:00Z",
        "endDate": "2026-10-31T23:59:59Z",
        "status": "ACTIVE",
        "createdAt": "2026-10-13T10:00:00Z",
        "updatedAt": "2026-10-13T10:00:00Z"
      }
    ],
    "page": 0,
    "size": 20,
    "totalElements": 1,
    "totalPages": 1
  },
  "message": null
}
```

---

### 4.2 Create Coupon (Admin)

```http
POST /api/v1/admin/coupons
```

Request:

```json
{
  "code": "VIPTECH100",
  "name": "VIP Customer Voucher",
  "description": "Direct 100,000 VND discount for VIP campaign",
  "discountType": "FIXED_AMOUNT",
  "discountValue": 100000.0,
  "maxDiscountAmount": null,
  "minOrderValue": 2000000.0,
  "usageLimit": 500,
  "usageLimitPerUser": 1,
  "isPublic": false,
  "startDate": "2026-10-15T00:00:00Z",
  "endDate": "2026-11-15T23:59:59Z"
}
```

Response (`201 Created`):

```json
{
  "success": true,
  "data": {
    "id": 3,
    "code": "VIPTECH100",
    "name": "VIP Customer Voucher",
    "discountType": "FIXED_AMOUNT",
    "discountValue": 100000.0,
    "status": "ACTIVE"
  },
  "message": "Coupon created successfully"
}
```

---

### 4.3 Update Coupon (Admin)

Updates coupon metadata, dates, or quotas. Enforces financial immutability per `BR-COUPON-008`.

```http
PUT /api/v1/admin/coupons/{id}
```

Request:

```json
{
  "name": "Extended VIP Customer Voucher",
  "description": "Updated terms and extended expiration",
  "usageLimit": 800,
  "endDate": "2026-11-30T23:59:59Z"
}
```

Rules:
- If `used_count == 0`: Admin may adjust all fields.
- If `used_count > 0`: `code`, `discountType`, `discountValue`, `maxDiscountAmount`, and `minOrderValue` are strictly immutable. Only `name`, `description`, `endDate` (extension), and `usageLimit` (increase) may be updated.

Response (`200 OK`):

```json
{
  "success": true,
  "data": {
    "id": 3,
    "code": "VIPTECH100",
    "name": "Extended VIP Customer Voucher",
    "usageLimit": 800,
    "endDate": "2026-11-30T23:59:59Z",
    "status": "ACTIVE"
  },
  "message": "Coupon updated successfully"
}
```

Response (`400 Bad Request` — Violation of Immutability):

```json
{
  "success": false,
  "data": null,
  "message": "Financial terms (discountType, discountValue, minOrderValue) cannot be modified after redemptions have occurred",
  "errorCode": "COUPON_IMMUTABLE_FIELD_CHANGE"
}
```

---

### 4.4 Toggle Coupon Status (Admin)

```http
PATCH /api/v1/admin/coupons/{id}/status
```

Request:

```json
{
  "status": "INACTIVE"
}
```

Response (`200 OK`):

```json
{
  "success": true,
  "data": {
    "id": 3,
    "code": "VIPTECH100",
    "status": "INACTIVE"
  },
  "message": "Coupon status updated successfully"
}
```
