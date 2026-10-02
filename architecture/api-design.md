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

# 7. Authentication APIs

## 7.1 Register

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

## 7.2 Login

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

# 8. Product APIs

## 8.1 Get Products

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

## 8.2 Get Product Detail

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

# 9. Category APIs

## 9.1 Get Categories

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
```
