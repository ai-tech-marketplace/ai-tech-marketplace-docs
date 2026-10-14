# Product Review & Rating API Contract

## 1. Overview

Product Review APIs allow verified purchasers to submit ratings and feedback on technology products, and allow public users and the AI Assistant to consume product reputation metrics.

Base URL:

```text
/api/v1/products/{productId}/reviews
```

---

## 2. Get Product Reviews

### Endpoint

```http
GET /api/v1/products/{productId}/reviews
```

### Authentication

Public.

### Query Parameters

| Parameter | Required | Description |
| --- | --- | --- |
| `page` | No | Page number, default 0 |
| `size` | No | Page size, default 10 |
| `rating` | No | Filter by star rating (1–5) |

### Success Response

HTTP `200 OK`

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "id": 1,
        "authorName": "John D.",
        "rating": 5,
        "comment": "Laptop chạy Docker và compile Java cực mượt, tản nhiệt tốt, màn hình OLED rất đẹp.",
        "isVerifiedPurchase": true,
        "createdAt": "2026-10-10T14:30:00Z"
      }
    ],
    "page": 0,
    "size": 10,
    "totalElements": 1,
    "totalPages": 1
  },
  "message": null
}
```

---

## 3. Get Review Rating Summary

### Endpoint

```http
GET /api/v1/products/{productId}/reviews/summary
```

### Authentication

Public.

### Success Response

HTTP `200 OK`

```json
{
  "success": true,
  "data": {
    "productId": 1,
    "averageRating": 4.8,
    "totalReviews": 45,
    "ratingDistribution": {
      "5": 38,
      "4": 5,
      "3": 2,
      "2": 0,
      "1": 0
    }
  },
  "message": null
}
```

---

## 4. Submit Product Review

### Endpoint

```http
POST /api/v1/products/{productId}/reviews
```

### Authentication

Required (`ROLE_CUSTOMER`).

### Request

```json
{
  "orderItemId": 25,
  "rating": 5,
  "comment": "Máy chạy rất êm, đóng gói cẩn thận, pin dùng được khoảng 6 tiếng khi code."
}
```

### Validation

| Field | Required | Rules |
| --- | --- | --- |
| `orderItemId` | Yes | ID of delivered order item |
| `rating` | Yes | Integer between 1 and 5 |
| `comment` | No | Max 2000 characters |

### Business Rules

- **BR-REVIEW-001 (Verified Purchase):** Caller must have purchased this product in an order whose status is `DELIVERED`.
- **BR-REVIEW-003 (Uniqueness):** A customer can only write one review per purchased item. Attempting to submit a second review returns `409 Conflict`.

### Success Response

HTTP `201 Created`

```json
{
  "success": true,
  "data": {
    "id": 2,
    "authorName": "John D.",
    "rating": 5,
    "comment": "Máy chạy rất êm, đóng gói cẩn thận, pin dùng được khoảng 6 tiếng khi code.",
    "isVerifiedPurchase": true,
    "createdAt": "2026-10-13T13:30:00Z"
  },
  "message": "Review submitted successfully"
}
```
