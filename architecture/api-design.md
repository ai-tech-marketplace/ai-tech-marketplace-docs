# API Design

## 1. API Overview

The AI Tech Marketplace backend exposes REST APIs for:

* Authentication
* Users
* Products
* Categories
* Brands
* Cart
* Orders
* Inventory
* AI Shopping Assistant
* Administration

The API uses JSON for request and response payloads.

## 2. Base URL

Development:

```text
http://localhost:8080/api/v1
```

Production:

```text
https://api.example.com/api/v1
```

The production domain is a placeholder and will be configured during deployment.

## 3. API Versioning

The API uses URL-based versioning.

Example:

```text
/api/v1/products
/api/v1/orders
/api/v1/ai/chat
```

Future breaking changes can use:

```text
/api/v2/...
```

## 4. Authentication

Authentication uses JWT.

Protected requests should include:

```http
Authorization: Bearer <access_token>
```

Example:

```http
GET /api/v1/orders
Authorization: Bearer eyJ...
```

## 5. Standard Response Format

Successful response:

```json
{
  "success": true,
  "data": {},
  "message": null
}
```

Error response:

```json
{
  "success": false,
  "data": null,
  "message": "Product not found",
  "errorCode": "PRODUCT_NOT_FOUND"
}
```

## 6. HTTP Status Codes

The API uses standard HTTP status codes.

| Status | Meaning                                  |
| ------ | ---------------------------------------- |
| 200    | Successful request                       |
| 201    | Resource created                         |
| 204    | Successful request with no response body |
| 400    | Invalid request                          |
| 401    | Authentication required or invalid       |
| 403    | Insufficient permission                  |
| 404    | Resource not found                       |
| 409    | Conflict                                 |
| 422    | Business validation error                |
| 500    | Internal server error                    |
| 503    | Service temporarily unavailable          |

## 7. Authentication APIs

### 7.1 Register

```http
POST /api/v1/auth/register
```

Request:

```json
{
  "email": "customer@example.com",
  "password": "Password123!",
  "fullName": "John Doe",
  "phone": "0900000000"
}
```

Response:

```json
{
  "success": true,
  "data": {
    "id": 1,
    "email": "customer@example.com",
    "fullName": "John Doe",
    "role": "CUSTOMER"
  },
  "message": "Registration successful"
}
```

### 7.2 Login

```http
POST /api/v1/auth/login
```

Request:

```json
{
  "email": "customer@example.com",
  "password": "Password123!"
}
```

Response:

```json
{
  "success": true,
  "data": {
    "accessToken": "jwt-token",
    "tokenType": "Bearer",
    "expiresIn": 3600,
    "user": {
      "id": 1,
      "email": "customer@example.com",
      "fullName": "John Doe",
      "role": "CUSTOMER"
    }
  },
  "message": "Login successful"
}
```

## 8. Product APIs

### 8.1 Get Products

```http
GET /api/v1/products
```

Query parameters:

```text
page
size
keyword
categoryId
brandId
minPrice
maxPrice
sort
```

Example:

```http
GET /api/v1/products?page=0&size=20&keyword=laptop&minPrice=20000000&maxPrice=35000000
```

Response:

```json
{
  "success": true,
  "data": {
    "content": [
      {
        "id": 1,
        "sku": "LAP-001",
        "name": "Gaming Laptop",
        "price": 29990000,
        "brand": {
          "id": 1,
          "name": "ASUS"
        },
        "category": {
          "id": 1,
          "name": "Laptop"
        },
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

### 8.2 Get Product Detail

```http
GET /api/v1/products/{id}
```

Example:

```http
GET /api/v1/products/1
```

Response:

```json
{
  "success": true,
  "data": {
    "id": 1,
    "sku": "LAP-001",
    "name": "Gaming Laptop",
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
      "storage": "1TB SSD"
    },
    "inventory": {
      "available": 10
    }
  },
  "message": null
}
```

## 9. Category APIs

### 9.1 Get Categories

```http
GET /api/v1/categories
```

Response:

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "Laptop",
      "slug": "laptop",
      "description": "Laptops for work, development, and gaming"
    },
    {
      "id": 2,
      "name": "Smartphone",
      "slug": "smartphone",
      "description": "Mobile phones and accessories"
    }
  ],
  "message": null
}
```

### 9.2 Get Category Detail

```http
GET /api/v1/categories/{id}
```

Response:

```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "Laptop",
    "slug": "laptop",
    "description": "Laptops for work, development, and gaming"
  },
  "message": null
}
```

---

## 10. Brand APIs

### 10.1 Get Brands

```http
GET /api/v1/brands
```

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

## 11. Shopping Cart APIs

Shopping Cart endpoints manage customer selections prior to checkout. All endpoints require authentication.

For the complete API contract, see [Cart API Contract](../api/cart.md).

Summary of endpoints:

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/api/v1/cart` | Get current customer's active cart |
| `POST` | `/api/v1/cart/items` | Add product to cart |
| `PUT` | `/api/v1/cart/items/{itemId}` | Update quantity of a cart item |
| `DELETE` | `/api/v1/cart/items/{itemId}` | Remove an item from cart |
| `DELETE` | `/api/v1/cart/items` | Clear all items from cart |

Example response (`GET /api/v1/cart`):

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

## 12. Order Management APIs

Order APIs handle checkout, order creation, order tracking, and cancellation.

For the complete API contract, see [Order API Contract](../api/orders.md).

Summary of endpoints:

| Method | Endpoint | Description | Access |
| --- | --- | --- | --- |
| `POST` | `/api/v1/orders` | Create order from active cart | Customer |
| `GET` | `/api/v1/orders` | List customer's orders | Customer |
| `GET` | `/api/v1/orders/{id}` | Get order details | Customer / Admin |
| `POST` | `/api/v1/orders/{id}/cancel` | Cancel an order | Customer / Admin |

Example request (`POST /api/v1/orders`):

```json
{
  "addressId": 10
}
```

---

## 13. Inventory APIs

Inventory APIs provide stock availability information and administrative inventory adjustment.

Summary of endpoints:

| Method | Endpoint | Description | Access |
| --- | --- | --- | --- |
| `GET` | `/api/v1/inventory/{productId}` | Get real-time stock availability | Public / Authenticated |
| `PUT` | `/api/v1/admin/inventory/{productId}` | Update stock quantity | Admin / Seller |

Example response (`GET /api/v1/inventory/1`):

```json
{
  "success": true,
  "data": {
    "productId": 1,
    "availableQuantity": 15,
    "isInStock": true
  },
  "message": null
}
```

---

## 14. AI Shopping Assistant APIs

AI Shopping Assistant endpoints provide conversational product recommendations, comparisons, and semantic search.

For the complete API contract, see [AI API Contract](../api/ai.md).

Summary of endpoints:

| Method | Endpoint | Description | Access |
| --- | --- | --- | --- |
| `POST` | `/api/v1/ai/chat` | Conversational shopping assistant | Authenticated |
| `POST` | `/api/v1/ai/recommendations` | Structured requirement-based recommendations | Authenticated |
| `POST` | `/api/v1/ai/compare` | Specification and feature comparison | Authenticated |

Example request (`POST /api/v1/ai/chat`):

```json
{
  "message": "Tôi cần laptop khoảng 30 triệu để lập trình Java và chạy Docker.",
  "conversationId": null
}
```

---

## 15. Administration APIs

Administrative endpoints for catalog and user governance require the `ADMIN` role.

| Method | Endpoint | Description |
| --- | --- | --- |
| `POST` | `/api/v1/admin/products` | Create a new product |
| `PUT` | `/api/v1/admin/products/{id}` | Update product information |
| `DELETE` | `/api/v1/admin/products/{id}` | Soft-delete / deactivate product |
| `POST` | `/api/v1/admin/categories` | Create product category |
| `PUT` | `/api/v1/admin/categories/{id}` | Update product category |
| `GET` | `/api/v1/admin/orders` | View all platform orders |
| `PUT` | `/api/v1/admin/orders/{id}/status` | Update order fulfillment status |

---

## 16. Detailed Contract Directory

The individual API contracts are maintained in dedicated documents:

| Domain | Contract Document | Responsibility |
| --- | --- | --- |
| Authentication | [authentication.md](../api/authentication.md) | Register, Login, Refresh token, Logout, Current user |
| Products | [products.md](../api/products.md) | Catalog, Search, Filtering, Categories, Brands |
| Shopping Cart | [cart.md](../api/cart.md) | Cart items, Stock validation, Subtotal |
| Orders | [orders.md](../api/orders.md) | Order creation, Status transitions, Events |
| AI Shopping Assistant | [ai.md](../api/ai.md) | Chat, Recommendation, Comparison, RAG |
