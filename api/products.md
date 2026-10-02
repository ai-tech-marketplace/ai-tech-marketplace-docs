# Product API Contract

## 1. Overview

Product APIs provide product discovery and product detail functionality.

Base URL:

```text
/api/v1/products
```

---

# 2. Get Products

## Endpoint

```http
GET /api/v1/products
```

## Authentication

Public

## Query Parameters

| Parameter  | Required | Description            |
| ---------- | -------- | ---------------------- |
| page       | No       | Page number, default 0 |
| size       | No       | Page size, default 20  |
| keyword    | No       | Search keyword         |
| categoryId | No       | Filter by category     |
| brandId    | No       | Filter by brand        |
| minPrice   | No       | Minimum price          |
| maxPrice   | No       | Maximum price          |
| sort       | No       | Sorting                |

Example:

```http
GET /api/v1/products?page=0&size=20&keyword=laptop&minPrice=20000000&maxPrice=35000000
```

## Response

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "id": 1,
        "sku": "LAP-001",
        "name": "Gaming Laptop",
        "slug": "gaming-laptop",
        "price": 29990000,
        "brand": {
          "id": 1,
          "name": "ASUS"
        },
        "category": {
          "id": 1,
          "name": "Laptop"
        },
        "thumbnailUrl": "/images/laptop-001.jpg",
        "status": "ACTIVE"
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

# 3. Get Product Detail

## Endpoint

```http
GET /api/v1/products/{id}
```

## Authentication

Public

## Response

```json
{
  "success": true,
  "data": {
    "id": 1,
    "sku": "LAP-001",
    "name": "Gaming Laptop",
    "slug": "gaming-laptop",
    "description": "Gaming and development laptop",
    "price": 29990000,
    "brand": {
      "id": 1,
      "name": "ASUS"
    },
    "category": {
      "id": 1,
      "name": "Laptop"
    },
    "specifications": {
      "cpu": "Intel Core Ultra 9",
      "gpu": "RTX 5070",
      "ram": "32GB",
      "storage": "1TB SSD",
      "display": "16 inch OLED"
    },
    "inventory": {
      "available": 10
    },
    "images": [
      "/images/laptop-001-1.jpg",
      "/images/laptop-001-2.jpg"
    ]
  },
  "message": null
}
```

---

# 4. Search Products

The normal product endpoint supports keyword search.

Example:

```http
GET /api/v1/products?keyword=gaming+laptop
```

The initial implementation may use PostgreSQL search.

Advanced semantic search will be handled by the AI service later.

---

# 5. Categories

## Get Categories

```http
GET /api/v1/categories
```

Public.

Response:

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Laptop",
      "slug": "laptop"
    },
    {
      "id": 2,
      "name": "Smartphone",
      "slug": "smartphone"
    },
    {
      "id": 3,
      "name": "Monitor",
      "slug": "monitor"
    }
  ],
  "message": null
}
```

---

# 6. Brands

## Get Brands

```http
GET /api/v1/brands
```

Public.

Response:

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "ASUS",
      "slug": "asus"
    },
    {
      "id": 2,
      "name": "Lenovo",
      "slug": "lenovo"
    }
  ],
  "message": null
}
```

---

# 7. Admin Product APIs

Admin authentication required.

## Create Product

```http
POST /api/v1/admin/products
```

Request:

```json
{
  "sku": "LAP-002",
  "name": "Developer Laptop",
  "description": "Laptop for software development",
  "price": 25990000,
  "categoryId": 1,
  "brandId": 2,
  "specifications": {
    "cpu": "Intel Core Ultra 7",
    "ram": "32GB",
    "storage": "1TB SSD"
  }
}
```

## Update Product

```http
PUT /api/v1/admin/products/{id}
```

## Delete Product

```http
DELETE /api/v1/admin/products/{id}
```

Deletion should normally be implemented as a soft delete or status change rather than physically deleting historical product data.

---

# 8. Product Status

Initial values:

```text
ACTIVE
INACTIVE
OUT_OF_STOCK
```

The backend is responsible for determining valid status transitions.

---

# 9. Product Ownership

Product data is owned by the backend.

```text
React
  |
  v
Backend
  |
  v
PostgreSQL
```

The AI service may consume product information but does not own the transactional product data.
