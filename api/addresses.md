# Address API Contract

## 1. Overview

Address APIs manage customer shipping addresses used during checkout.

Base URL:

```text
/api/v1/addresses
```

All endpoints require authentication. Customers can only view and manage their own addresses.

---

## 2. Get Customer Addresses

### Endpoint

```http
GET /api/v1/addresses
```

### Authentication

Required (`Authorization: Bearer <token>`).

### Success Response

HTTP `200 OK`

```json
{
  "success": true,
  "data": [
    {
      "id": 10,
      "recipientName": "John Doe",
      "phone": "0900000000",
      "addressLine": "123 Example Street",
      "ward": "Dich Vong Hau",
      "district": "Cau Giay",
      "city": "Hanoi",
      "country": "Vietnam",
      "isDefault": true
    },
    {
      "id": 11,
      "recipientName": "John Doe (Office)",
      "phone": "0900000001",
      "addressLine": "456 Office Tower",
      "ward": "Ben Nghe",
      "district": "District 1",
      "city": "Ho Chi Minh City",
      "country": "Vietnam",
      "isDefault": false
    }
  ],
  "message": null
}
```

---

## 3. Create Shipping Address

### Endpoint

```http
POST /api/v1/addresses
```

### Request

```json
{
  "recipientName": "John Doe",
  "phone": "0900000000",
  "addressLine": "123 Example Street",
  "ward": "Dich Vong Hau",
  "district": "Cau Giay",
  "city": "Hanoi",
  "country": "Vietnam",
  "isDefault": true
}
```

### Validation

| Field | Required | Rules |
| --- | --- | --- |
| `recipientName` | Yes | 2–100 characters |
| `phone` | Yes | Valid phone format (10–15 digits) |
| `addressLine` | Yes | 5–255 characters |
| `ward` | No | Max 100 characters |
| `district` | No | Max 100 characters |
| `city` | Yes | Max 100 characters |
| `country` | Yes | Max 100 characters, defaults to "Vietnam" |
| `isDefault` | No | Boolean, defaults to false (if first address, automatically true) |

### Success Response

HTTP `201 Created`

```json
{
  "success": true,
  "data": {
    "id": 10,
    "recipientName": "John Doe",
    "phone": "0900000000",
    "addressLine": "123 Example Street",
    "ward": "Dich Vong Hau",
    "district": "Cau Giay",
    "city": "Hanoi",
    "country": "Vietnam",
    "isDefault": true
  },
  "message": "Address created successfully"
}
```

---

## 4. Update Shipping Address

### Endpoint

```http
PUT /api/v1/addresses/{id}
```

### Request

```json
{
  "recipientName": "John Doe",
  "phone": "0911222333",
  "addressLine": "123 Example Street, Suite 400",
  "ward": "Dich Vong Hau",
  "district": "Cau Giay",
  "city": "Hanoi",
  "country": "Vietnam",
  "isDefault": false
}
```

### Success Response

HTTP `200 OK`

```json
{
  "success": true,
  "data": {
    "id": 10,
    "recipientName": "John Doe",
    "phone": "0911222333",
    "addressLine": "123 Example Street, Suite 400",
    "ward": "Dich Vong Hau",
    "district": "Cau Giay",
    "city": "Hanoi",
    "country": "Vietnam",
    "isDefault": false
  },
  "message": "Address updated successfully"
}
```

---

## 5. Set Default Address

### Endpoint

```http
PATCH /api/v1/addresses/{id}/default
```

### Success Response

HTTP `200 OK`

```json
{
  "success": true,
  "data": {
    "id": 10,
    "isDefault": true
  },
  "message": "Default address updated"
}
```

---

## 6. Delete Shipping Address

### Endpoint

```http
DELETE /api/v1/addresses/{id}
```

### Business Rules

- A customer can only delete an address they own.
- If deleting an address that is currently marked `isDefault`, the most recently created remaining address is automatically assigned as default.
- If the address is referenced in existing historical orders, foreign key constraints must NOT cascade delete historical order shipping details (the order table snapshots shipping details as immutable text columns).

### Success Response

HTTP `204 No Content`
