# AI API Contract

## 1. Overview

The AI service provides intelligent shopping functionality.

Initial capabilities:

* AI Shopping Assistant
* Product recommendation
* Product comparison
* Natural language product search
* Semantic search
* RAG
* LLM response generation

---

## 2. Architecture

The frontend does not communicate directly with the AI service.

```text
React
  |
  | POST /api/v1/ai/chat
  v
Spring Boot
  |
  | Internal API
  v
Python FastAPI
  |
  ├── Retrieval
  ├── Embedding
  ├── Recommendation
  ├── RAG
  └── LLM
```

---

## 3. AI Chat

### Public Backend Endpoint

```http
POST /api/v1/ai/chat
```

Authentication:

Required.

### Request

```json
{
  "message": "Tôi cần laptop khoảng 30 triệu để code Java và chạy Docker.",
  "conversationId": null
}
```

For an existing conversation:

```json
{
  "message": "Có máy nào nhẹ hơn không?",
  "conversationId": "conv-123"
}
```

---

## 4. AI Chat Response

```json
{
  "success": true,
  "data": {
    "conversationId": "conv-123",
    "message": "Với nhu cầu Java development và Docker, bạn nên ưu tiên CPU mạnh, RAM từ 32GB và SSD tối thiểu 1TB.",
    "products": [
      {
        "id": 101,
        "name": "Developer Laptop A",
        "price": 28990000,
        "reason": "32GB RAM và CPU phù hợp cho development"
      },
      {
        "id": 102,
        "name": "Developer Laptop B",
        "price": 30990000,
        "reason": "Hiệu năng tốt cho Docker và đa nhiệm"
      }
    ]
  },
  "message": null
}
```

---

## 5. AI Recommendation

### Endpoint

```http
POST /api/v1/ai/recommendations
```

Request:

```json
{
  "category": "LAPTOP",
  "budget": 30000000,
  "useCase": [
    "JAVA_DEVELOPMENT",
    "DOCKER"
  ],
  "preferences": {
    "minimumRam": 32,
    "minimumStorage": 1000
  }
}
```

Response:

```json
{
  "success": true,
  "data": {
    "recommendations": [
      {
        "productId": 101,
        "score": 0.94,
        "reason": "32GB RAM and strong CPU for development"
      },
      {
        "productId": 102,
        "score": 0.89,
        "reason": "Good balance between performance and price"
      }
    ]
  },
  "message": null
}
```

---

## 6. Product Comparison

### Endpoint

```http
POST /api/v1/ai/compare
```

Request:

```json
{
  "productIds": [
    101,
    102
  ],
  "criteria": [
    "performance",
    "battery",
    "portability",
    "price"
  ]
}
```

Response:

```json
{
  "success": true,
  "data": {
    "products": [
      {
        "productId": 101,
        "strengths": [
          "Strong CPU",
          "32GB RAM"
        ],
        "weaknesses": [
          "Heavier"
        ]
      },
      {
        "productId": 102,
        "strengths": [
          "Better battery",
          "Lighter"
        ],
        "weaknesses": [
          "Lower GPU performance"
        ]
      }
    ],
    "summary": "The products prioritize different use cases."
  },
  "message": null
}
```

---

## 7. Internal AI API

The backend communicates with Python FastAPI through internal endpoints.

Example:

```http
POST /internal/ai/chat
```

Request:

```json
{
  "userId": 1,
  "conversationId": "conv-123",
  "message": "Laptop khoảng 30 triệu để code Java",
  "context": {
    "category": "LAPTOP",
    "budget": 30000000
  }
}
```

---

## 8. AI Retrieval

The AI service may request product information from the backend.

Possible architecture:

```text
AI Service
    |
    | Product Search API
    v
Backend
    |
    v
PostgreSQL
```

The AI service must not directly modify transactional product data.

---

## 9. RAG Pipeline

Initial RAG architecture:

```text
User Question
      |
      v
Query Understanding
      |
      v
Embedding
      |
      v
Vector Search
      |
      v
Relevant Product Information
      |
      v
Context Construction
      |
      v
LLM
      |
      v
AI Response
```

---

## 10. Product Embeddings

Products may be transformed into searchable documents.

Example:

```text
Product:
ASUS Developer Laptop

Category:
Laptop

Specifications:
Intel Core Ultra 9
32GB RAM
1TB SSD
RTX 5070

Use Cases:
Java Development
Docker
Gaming
```

This information can be converted into an embedding.

---

## 11. Recommendation

Recommendation may initially use rule-based filtering.

Example:

```text
User:
Budget = 30M
RAM >= 32GB
Use case = Java + Docker

        |
        v

Filter products
        |
        v

Calculate recommendation score
        |
        v

Return top products
```

AI/ML-based recommendation can be introduced later.

---

## 12. LLM Responsibility

The LLM should primarily be responsible for:

* Understanding natural language
* Extracting user requirements
* Generating explanations
* Summarizing retrieved product information
* Conversational interaction

The LLM should not be the source of truth for:

* Product price
* Inventory
* Order status
* User permissions

Those values must come from backend systems.

---

## 13. AI Hallucination Prevention

The AI response should be grounded in retrieved product data.

The system should avoid allowing the LLM to invent:

* Product specifications
* Prices
* Inventory
* Discounts
* Order information

Example:

```text
Database
    |
    | Grounded product data
    v
Retrieval
    |
    v
LLM
    |
    v
Response
```

---

## 14. AI Failure Handling

If the AI service is unavailable:

```text
React
  |
  v
Backend
  |
  X
AI Service
```

The backend should return:

```json
{
  "success": false,
  "data": null,
  "message": "AI assistant is temporarily unavailable",
  "errorCode": "AI_SERVICE_UNAVAILABLE"
}
```

The marketplace itself must remain functional.

Users should still be able to:

* Browse products
* Search products
* Add products to cart
* Create orders
* View orders

---

## 15. AI Security

The AI service must not expose:

* LLM API keys
* Database credentials
* Internal service credentials

Sensitive configuration must be provided through environment variables or a secure secret management system.

---

## 16. AI Rate Limiting

AI endpoints may have stricter rate limits than normal product APIs.

Example:

```text
Normal API:
100 requests/minute

AI API:
20 requests/minute
```

The exact limits will be configured later.

---

## 17. Conversation

The AI assistant supports conversations.

Example:

```text
User:
Tôi cần laptop 30 triệu.

AI:
Bạn ưu tiên hiệu năng hay tính di động?

User:
Tôi ưu tiên hiệu năng.

AI:
Tôi sẽ tập trung vào CPU, RAM và GPU...
```

The backend should maintain a conversation identifier.

```text
conversationId
      |
      ├── Message 1
      ├── Message 2
      ├── Message 3
      └── Message 4
```

Conversation persistence strategy will be defined during AI implementation.

---

## 18. AI API Ownership

Public AI API:

```text
React
  |
  v
Spring Boot
```

Internal AI API:

```text
Spring Boot
  |
  v
FastAPI
```

This creates a clear security boundary.

---

## 19. Future AI Capabilities

Potential future capabilities:

* Personalized recommendations
* User preference learning
* Product review summarization
* Review sentiment analysis
* Voice shopping assistant
* Image-based product search
* AI-generated product descriptions
* Semantic product search
* Shopping agent workflows
