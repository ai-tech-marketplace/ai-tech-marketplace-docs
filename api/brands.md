# Brand API Contract

## 1. Overview

Brand APIs provide public brand discovery and administrative brand management.

Base URL:

```text
/api/v1/brands
```

---

## 2. Get Brands

### Endpoint

```http
GET /api/v1/brands
```

### Authentication

Public.

### Success Response

HTTP `200 OK`

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "ASUS",
      "slug": "asus",
      "status": "ACTIVE"
    },
    {
      "id": 2,
      "name": "Lenovo",
      "slug": "lenovo",
      "status": "ACTIVE"
    },
    {
      "id": 3,
      "name": "Apple",
      "slug": "apple",
      "status": "ACTIVE"
    },
    {
      "id": 4,
      "name": "Dell",
      "slug": "dell",
      "status": "ACTIVE"
    }
  ],
  "message": null
}
```

---

## 3. Get Brand Detail

### Endpoint

```http
GET /api/v1/brands/{id}
```

### Authentication

Public.

### Success Response

HTTP `200 OK`

```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "ASUS",
    "slug": "asus",
    "status": "ACTIVE"
  },
  "message": null
}
```

---

## 4. Admin Create Brand

### Endpoint

```http
POST /api/v1/admin/brands
```

### Authentication

Required (`ROLE_ADMIN`).

### Request

```json
{
  "name": "Razer",
  "slug": "razer"
}
```

### Validation

| Field | Required | Rules |
| --- | --- | --- |
| `name` | Yes | 1–100 characters, unique |
| `slug` | Yes | 1–150 characters, URL-friendly format, unique |

### Success Response

HTTP `201 Created`

```json
{
  "success": true,
  "data": {
    "id": 5,
    "name": "Razer",
    "slug": "razer",
    "status": "ACTIVE"
  },
  "message": "Brand created successfully"
}
```

---

## 5. Admin Update Brand

### Endpoint

```http
PUT /api/v1/admin/brands/{id}
```

### Authentication

Required (`ROLE_ADMIN`).

### Request

```json
{
  "name": "Razer Gaming",
  "slug": "razer-gaming",
  "status": "ACTIVE"
}
```

### Success Response

HTTP `200 OK`

```json
{
  "success": true,
  "data": {
    "id": 5,
    "name": "Razer Gaming",
    "slug": "razer-gaming",
    "status": "ACTIVE"
  },
  "message": "Brand updated successfully"
}
```

---

## 6. Admin Deactivate Brand

### Endpoint

```http
DELETE /api/v1/admin/brands/{id}
```

### Authentication

Required (`ROLE_ADMIN`).

### Business Rules

- Brands are soft-deactivated (`status = 'INACTIVE'`) to preserve historical integrity for existing products.
- Products associated with inactive brands remain visible in historical orders.

### Success Response

HTTP `204 No Content`
