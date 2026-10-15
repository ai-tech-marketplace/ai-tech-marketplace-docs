# Commission Settings API Contract

## 1. Overview

The Commission Settings API allows platform administrators to configure, inspect, and update commission rates applied to sales across the marketplace.

Base URL:

```text
/api/v1/admin/commission-settings
```

Access Requirement:
- `ROLE_ADMIN`

---

## 2. Error Codes

| Error Code | HTTP Status | Description |
| --- | --- | --- |
| `COMMISSION_NOT_FOUND` | 404 | Commission configuration record does not exist |
| `COMMISSION_INVALID_RATE` | 400 | Commission rate must be between 0.0000 and 1.0000 (0% to 100%) |
| `COMMISSION_DUPLICATE_SCOPE` | 409 | A configuration for this `(scope, target_id)` already exists |
| `COMMISSION_TARGET_NOT_FOUND` | 404 | Specified `targetId` (category or seller) does not exist |
| `COMMISSION_INVALID_SCOPE` | 400 | Scope must be `GLOBAL`, `CATEGORY`, or `SELLER` |

---

## 3. Endpoints

### 3.1 List Commission Configurations

Retrieve all commission rules across global, category, and seller scopes.

```http
GET /api/v1/admin/commission-settings
```

Headers:
- `Authorization`: `Bearer <admin_jwt_token>`

Query Parameters:
- `scope`: Optional filter (`GLOBAL`, `CATEGORY`, `SELLER`)
- `isActive`: Optional filter (`true`, `false`)

Response (`200 OK`):

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "scope": "GLOBAL",
      "targetId": null,
      "targetName": null,
      "rate": 0.0500,
      "description": "Standard platform default commission (5%)",
      "isActive": true,
      "createdAt": "2026-10-15T08:00:00Z",
      "updatedAt": "2026-10-15T08:00:00Z"
    },
    {
      "id": 2,
      "scope": "CATEGORY",
      "targetId": 1,
      "targetName": "Laptops & Notebooks",
      "rate": 0.0250,
      "description": "Reduced commission for high-ticket laptops (2.5%)",
      "isActive": true,
      "createdAt": "2026-10-15T08:30:00Z",
      "updatedAt": "2026-10-15T08:30:00Z"
    },
    {
      "id": 3,
      "scope": "CATEGORY",
      "targetId": 5,
      "targetName": "Accessories & Peripherals",
      "rate": 0.0800,
      "description": "Commission for accessories & cables (8%)",
      "isActive": true,
      "createdAt": "2026-10-15T08:35:00Z",
      "updatedAt": "2026-10-15T08:35:00Z"
    },
    {
      "id": 4,
      "scope": "SELLER",
      "targetId": 12,
      "targetName": "Asus Official Flagship Store",
      "rate": 0.0200,
      "description": "Special merchant agreement (2%)",
      "isActive": true,
      "createdAt": "2026-10-15T09:00:00Z",
      "updatedAt": "2026-10-15T09:00:00Z"
    }
  ],
  "error": null,
  "timestamp": "2026-10-15T09:30:00Z"
}
```

---

### 3.2 Create Commission Configuration

Define a new commission rate for a global, category, or seller target.

```http
POST /api/v1/admin/commission-settings
```

Headers:
- `Authorization`: `Bearer <admin_jwt_token>`
- `Content-Type`: `application/json`

Request Body:

```json
{
  "scope": "CATEGORY",
  "targetId": 2,
  "rate": 0.0350,
  "description": "Mobile phones category commission rate (3.5%)",
  "isActive": true
}
```

Field Constraints:
- `scope`: `VARCHAR`, Required, must be one of `GLOBAL`, `CATEGORY`, `SELLER`.
- `targetId`: `BIGINT`, Required if scope is `CATEGORY` or `SELLER`; must be `null` if scope is `GLOBAL`.
- `rate`: `NUMERIC(5,4)`, Required, $0.0000 \le \text{rate} \le 1.0000$.
- `description`: `VARCHAR(255)`, Optional.
- `isActive`: `BOOLEAN`, Default `true`.

Response (`201 Created`):

```json
{
  "success": true,
  "data": {
    "id": 5,
    "scope": "CATEGORY",
    "targetId": 2,
    "targetName": "Smartphones & Tablets",
    "rate": 0.0350,
    "description": "Mobile phones category commission rate (3.5%)",
    "isActive": true,
    "createdAt": "2026-10-15T10:00:00Z",
    "updatedAt": "2026-10-15T10:00:00Z"
  },
  "error": null,
  "timestamp": "2026-10-15T10:00:00Z"
}
```

---

### 3.3 Update Commission Configuration

Update rate percentage, status, or description of an existing rule.

```http
PUT /api/v1/admin/commission-settings/{id}
```

Headers:
- `Authorization`: `Bearer <admin_jwt_token>`
- `Content-Type`: `application/json`

Request Body:

```json
{
  "rate": 0.0400,
  "description": "Adjusted smartphone rate to 4.0%",
  "isActive": true
}
```

Response (`200 OK`):

```json
{
  "success": true,
  "data": {
    "id": 5,
    "scope": "CATEGORY",
    "targetId": 2,
    "targetName": "Smartphones & Tablets",
    "rate": 0.0400,
    "description": "Adjusted smartphone rate to 4.0%",
    "isActive": true,
    "createdAt": "2026-10-15T10:00:00Z",
    "updatedAt": "2026-10-15T10:15:00Z"
  },
  "error": null,
  "timestamp": "2026-10-15T10:15:00Z"
}
```

---

### 3.4 Delete / Deactivate Commission Configuration

Delete or deactivate a commission rule (reverts affected items to lower-precedence fallback rate).

```http
DELETE /api/v1/admin/commission-settings/{id}
```

Headers:
- `Authorization`: `Bearer <admin_jwt_token>`

Response (`200 OK`):

```json
{
  "success": true,
  "data": {
    "message": "Commission configuration successfully removed"
  },
  "error": null,
  "timestamp": "2026-10-15T10:20:00Z"
}
```
