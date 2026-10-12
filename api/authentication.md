# Authentication API Contract

## 1. Overview

This document defines the authentication API contract between the React frontend and Spring Boot backend.

Base URL:

```text
/api/v1/auth
```

Authentication endpoints:

```text
POST /register
POST /login
POST /refresh
POST /logout
GET  /me
```

---

## 2. Register

### Endpoint

```http
POST /api/v1/auth/register
```

### Authentication

Public

### Request

```json
{
  "email": "customer@example.com",
  "password": "Password123!",
  "fullName": "John Doe",
  "phone": "0900000000"
}
```

### Validation

| Field    | Required | Rules                |
| -------- | -------- | -------------------- |
| email    | Yes      | Valid email, unique  |
| password | Yes      | Minimum 8 characters |
| fullName | Yes      | 2–100 characters     |
| phone    | No       | Valid phone format   |

### Success Response

HTTP `201 Created`

```json
{
  "success": true,
  "data": {
    "id": 1,
    "email": "customer@example.com",
    "fullName": "John Doe",
    "phone": "0900000000",
    "role": "CUSTOMER"
  },
  "message": "Registration successful"
}
```

### Errors

#### Email already exists

HTTP `409 Conflict`

```json
{
  "success": false,
  "data": null,
  "message": "Email already exists",
  "errorCode": "USER_ALREADY_EXISTS"
}
```

---

## 3. Login

### Endpoint

```http
POST /api/v1/auth/login
```

### Authentication

Public

### Request

```json
{
  "email": "customer@example.com",
  "password": "Password123!"
}
```

### Success Response

HTTP `200 OK`

```json
{
  "success": true,
  "data": {
    "accessToken": "jwt-access-token",
    "refreshToken": "refresh-token",
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

### Invalid Credentials

HTTP `401 Unauthorized`

```json
{
  "success": false,
  "data": null,
  "message": "Invalid email or password",
  "errorCode": "AUTH_INVALID_CREDENTIALS"
}
```

---

## 4. Refresh Token

### Endpoint

```http
POST /api/v1/auth/refresh
```

### Authentication

Public

### Request

```json
{
  "refreshToken": "refresh-token"
}
```

### Success Response

HTTP `200 OK`

```json
{
  "success": true,
  "data": {
    "accessToken": "new-access-token",
    "refreshToken": "new-refresh-token",
    "tokenType": "Bearer",
    "expiresIn": 3600
  },
  "message": null
}
```

---

## 5. Logout

### Endpoint

```http
POST /api/v1/auth/logout
```

### Authentication

Required

```http
Authorization: Bearer <access-token>
```

### Response

HTTP `204 No Content`

No response body is required.

---

## 6. Get Current User

### Endpoint

```http
GET /api/v1/auth/me
```

### Authentication

Required

### Response

HTTP `200 OK`

```json
{
  "success": true,
  "data": {
    "id": 1,
    "email": "customer@example.com",
    "fullName": "John Doe",
    "phone": "0900000000",
    "role": "CUSTOMER"
  },
  "message": null
}
```

---

## 7. Authentication Flow

```text
React
  |
  | POST /auth/login
  v
Backend
  |
  | Validate credentials
  v
PostgreSQL
  |
  | User
  v
Backend
  |
  | JWT
  v
React
```

Protected request:

```text
React
  |
  | Authorization: Bearer JWT
  v
Spring Security
  |
  | Validate token
  v
Controller
  |
  v
Service
```

---

## 8. Roles

Initial roles:

```text
CUSTOMER
SELLER
ADMIN
```

Role permissions will be defined separately.

---

## 9. Security Rules

The backend must:

* Hash passwords using a secure password hashing algorithm
* Never return password hashes
* Never log passwords
* Validate JWT signatures
* Validate JWT expiration
* Validate user status
* Protect admin endpoints with role-based authorization

The frontend must never store or expose backend secrets.
