# Sequence Diagrams

## 1. Overview

This document specifies the dynamic runtime interactions between components, services, data stores, and external systems for the critical workflows of AI Tech Marketplace.

---

## 2. Authentication & JWT Lifecycle Flow

```mermaid
sequenceDiagram
    autonumber
    actor Customer as Customer
    participant Frontend as React Frontend
    participant Backend as Spring Boot Backend
    participant DB as PostgreSQL

    Customer->>Frontend: Enters email & password
    Frontend->>Backend: POST /api/v1/auth/login
    activate Backend
    Backend->>DB: Query user by email (uk_users_email)
    DB-->>Backend: Return User (hash, is_active, roles)

    alt User not found or password mismatch
        Backend-->>Frontend: 401 Unauthorized (AUTH_INVALID_CREDENTIALS)
        Frontend-->>Customer: Display error message
    else Credentials valid & User active
        Backend->>Backend: Generate Access Token (1h) & Refresh Token (7d)
        Backend-->>Frontend: 200 OK (accessToken, refreshToken, user)
        deactivate Backend
        Frontend->>Frontend: Store tokens in secure memory / storage
        Frontend-->>Customer: Redirect to dashboard / catalog
    end

    opt Authenticated Request
        Customer->>Frontend: Navigates to protected view
        Frontend->>Backend: GET /api/v1/orders (Header: Authorization Bearer <token>)
        activate Backend
        Backend->>Backend: Verify JWT signature & expiration
        Backend->>DB: Query user-specific data
        DB-->>Backend: Return data
        Backend-->>Frontend: 200 OK (data payload)
        deactivate Backend
    end
```

---

## 3. Shopping Cart & Order Checkout Flow

```mermaid
sequenceDiagram
    autonumber
    actor Customer as Customer
    participant Frontend as React Frontend
    participant Backend as Spring Boot Backend
    participant DB as PostgreSQL
    participant Kafka as Kafka Broker
    participant Worker as Background Workers

    Customer->>Frontend: Clicks "Checkout" (selects address)
    Frontend->>Backend: POST /api/v1/orders { "addressId": 10 }
    activate Backend

    Note over Backend,DB: Transaction Boundary: @Transactional
    Backend->>DB: Load Customer Active Cart & CartItems
    DB-->>Backend: Return CartItems & Product data

    alt Cart is empty
        Backend-->>Frontend: 400 Bad Request ("Cart is empty")
    else Cart has items
        Backend->>DB: Validate stock availability (total - reserved >= quantity)
        
        alt Stock insufficient for any item
            Backend-->>Frontend: 422 Unprocessable Entity ("Insufficient stock")
        else Stock sufficient
            Backend->>Backend: Calculate immutable order total from current unit prices
            Backend->>DB: Insert into orders (status = PENDING, total_amount, address snapshot)
            Backend->>DB: Insert into order_items (snapshot name, sku, price, quantity)
            Backend->>DB: Update inventory (increment reserved_quantity)
            Backend->>DB: Clear Customer CartItems (DELETE cart_items)
            
            Backend->>Kafka: Publish Event OrderCreated (orderId, userId, items, total)
            deactivate Backend
            
            Backend-->>Frontend: 201 Created (Order Response Envelope)
            Frontend-->>Customer: Display Order Confirmation view
        end
    end

    par Order Notification Dispatch
        Kafka->>Worker: Consume OrderCreated
        activate Worker
        Worker->>Worker: Format order confirmation email
        Worker->>Customer: Send email notification
        deactivate Worker
    and Analytics & Sales Tracking
        Kafka->>Worker: Consume OrderCreated
        activate Worker
        Worker->>DB: Record analytics & sales event
        deactivate Worker
    end
```

---

## 4. AI Shopping Assistant RAG & Recommendation Flow

```mermaid
sequenceDiagram
    autonumber
    actor Customer as Customer
    participant Frontend as React Chat Widget
    participant Backend as Spring Boot Backend
    participant AI as Python FastAPI Service
    participant DB as PostgreSQL
    participant LLM as External LLM Provider

    Customer->>Frontend: "Tôi cần laptop 30tr để code Java và chạy Docker"
    Frontend->>Backend: POST /api/v1/ai/chat { message, conversationId }
    activate Backend

    Backend->>Backend: Validate JWT & apply AI rate limits
    Backend->>DB: Persist Customer Message (role=USER)
    
    Backend->>AI: Internal POST /internal/ai/chat { userId, message, conversationId }
    activate AI

    AI->>AI: Query Analysis & Requirement Extraction (Budget: 30M, RAM >= 32GB, CPU: High)
    AI->>LLM: Generate query embeddings
    LLM-->>AI: Return embedding vector

    AI->>Backend: GET /internal/products/search (Vector / Filter candidate lookup)
    Backend->>DB: Query active candidate products & specs
    DB-->>Backend: Return products
    Backend-->>AI: Return candidate list (specs, price, inventory)

    AI->>AI: Assemble RAG prompt with Grounded Product Catalog Context
    AI->>LLM: POST /chat/completions (System prompt, Grounded context, User query)
    activate LLM
    LLM-->>AI: Generated response with product recommendations & technical rationale
    deactivate LLM

    AI-->>Backend: Structured AI Payload (message, recommendedProducts, conversationId)
    deactivate AI

    Backend->>DB: Persist Assistant Message (role=ASSISTANT)
    Backend-->>Frontend: 200 OK ApiResponse { data: { message, products, conversationId } }
    deactivate Backend

    Frontend-->>Customer: Render conversational reply & interactive product recommendation cards
```

---

## 5. Summary of Sequence Patterns

| Workflow | Communication Type | Core Guarantees |
| --- | --- | --- |
| **Authentication** | Synchronous REST | Stateless JWT, BCrypt password hashing, role enforcement. |
| **Checkout & Orders** | Synchronous REST + Asynchronous Kafka | Database ACID transaction, inventory reservation concurrency control, at-least-once event delivery. |
| **AI Shopping Chat** | Synchronous Multi-tier REST (React → Spring Boot → FastAPI → LLM) | Private AI security boundary, grounded catalog retrieval (no spec hallucination), graceful fallback on AI outage. |
