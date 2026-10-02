# Architecture Decision Records

This document records important architecture decisions made during the development of AI Tech Marketplace.

---

# ADR-001 — Multi-Repository Architecture

## Status

Accepted

## Context

The project contains multiple applications with different responsibilities and technology stacks.

The main components are:

* React frontend
* Java Spring Boot backend
* Python AI service
* Infrastructure
* Documentation

## Decision

Use separate Git repositories for each major component.

```text
ai-tech-marketplace-backend
ai-tech-marketplace-frontend
ai-tech-marketplace-ai
ai-tech-marketplace-infra
ai-tech-marketplace-docs
```

## Reasons

* Clear separation of responsibilities
* Independent development
* Independent CI/CD pipelines
* Independent deployment
* Different technology stacks
* Easier repository ownership
* Easier service versioning

## Consequences

Advantages:

* Strong separation of concerns
* Independent deployments
* Clear project boundaries

Disadvantages:

* Cross-repository changes require coordination
* API contracts must be documented carefully
* More repositories to maintain

---

# ADR-002 — Spring Boot for Backend

## Status

Accepted

## Context

The core marketplace requires:

* Authentication
* Authorization
* Transactions
* REST APIs
* Database access
* Event-driven communication

## Decision

Use Java with Spring Boot.

## Reasons

Spring Boot provides mature support for:

* REST APIs
* Spring Security
* JPA
* Transaction management
* Validation
* Kafka
* Redis
* Testing

## Consequences

The backend will use the Spring ecosystem for core business services.

---

# ADR-003 — React for Frontend

## Status

Accepted

## Context

The application requires:

* Product browsing
* Search
* Filtering
* Cart
* Checkout
* Admin dashboard
* AI chat

## Decision

Use React with TypeScript.

## Reasons

* Component-based architecture
* Large ecosystem
* Strong TypeScript support
* Suitable for interactive applications
* Good integration with REST APIs

---

# ADR-004 — PostgreSQL as Primary Database

## Status

Accepted

## Context

Marketplace data requires transactional consistency.

Important operations include:

* Order creation
* Inventory management
* Cart operations
* User management

## Decision

Use PostgreSQL as the primary transactional database.

## Reasons

* ACID transactions
* Strong relational model
* Foreign key support
* JSONB support
* Mature ecosystem
* Good performance for transactional workloads

---

# ADR-005 — Redis for Cache and Distributed Coordination

## Status

Accepted

## Context

Some data requires low-latency access.

Examples:

* Frequently accessed product data
* Categories
* Temporary data
* Distributed locks

## Decision

Use Redis for:

* Caching
* Temporary data
* Distributed locking where appropriate

Redis will not be the source of truth for transactional data.

---

# ADR-006 — Kafka for Event-Driven Communication

## Status

Accepted

## Context

Some operations do not need to block the main request.

Examples:

* Notifications
* Analytics
* Inventory-related events
* Other background processing

## Decision

Use Kafka for asynchronous domain events.

Example:

```text
OrderCreated
OrderCancelled
InventoryReserved
InventoryReleased
ProductUpdated
```

## Consequences

Advantages:

* Loose coupling
* Asynchronous processing
* Independent consumers
* Better scalability

Disadvantages:

* More operational complexity
* Event ordering must be considered
* Consumers must handle duplicate events safely

---

# ADR-007 — Python FastAPI for AI Service

## Status

Accepted

## Context

AI functionality may require:

* LLM APIs
* Embeddings
* RAG
* Vector search
* Recommendation algorithms
* Machine learning libraries

## Decision

Implement the AI service using Python and FastAPI.

## Reasons

Python provides a strong ecosystem for:

* AI
* Machine learning
* NLP
* Embeddings
* LLM frameworks
* Data processing

FastAPI provides a lightweight HTTP API layer.

---

# ADR-008 — Backend as Business Authority

## Status

Accepted

## Context

The frontend and AI service may request marketplace operations.

Transactional operations must remain controlled.

## Decision

The backend is the authoritative service for marketplace business operations.

Examples:

* Creating orders
* Updating inventory
* Managing users
* Authorization
* Calculating order totals

The frontend and AI service cannot directly modify transactional database tables.

---

# ADR-009 — AI Behind Backend API

## Status

Accepted

## Context

The AI service could technically be exposed directly to the frontend.

However, this would create problems around:

* Authentication
* Authorization
* API key protection
* Rate limiting
* Business rules
* Service boundaries

## Decision

The frontend communicates with the backend.

The backend communicates with the AI service.

```text
React
  |
  v
Backend
  |
  v
AI Service
```

## Consequences

Advantages:

* Centralized authentication
* Better control
* AI service can remain private
* Easier monitoring
* Easier rate limiting

---

# ADR-010 — PostgreSQL JSONB for Product Specifications

## Status

Accepted

## Context

Technology products have different specifications.

For example:

Laptop:

```text
CPU
GPU
RAM
Storage
Display
```

Smartphone:

```text
CPU
RAM
Storage
Screen
Camera
Battery
```

Using a fixed relational table for every possible specification would create unnecessary complexity.

## Decision

Store flexible product specifications using PostgreSQL JSONB initially.

Example:

```json
{
  "cpu": "Intel Core Ultra 9",
  "gpu": "RTX 5070",
  "ram": "32GB",
  "storage": "1TB SSD"
}
```

## Consequences

Advantages:

* Flexible schema
* Easy to support multiple product categories
* Faster initial development

Disadvantages:

* Some queries are more complex
* Strong relational constraints cannot be applied to every JSON field

The specification model may be redesigned if search requirements become significantly more complex.

---

# ADR-011 — REST API for Synchronous Communication

## Status

Accepted

## Context

Frontend-to-backend operations require immediate responses.

Examples:

* Login
* Product browsing
* Cart operations
* Checkout

## Decision

Use REST APIs for synchronous communication.

Kafka will be used for asynchronous event processing.

```text
Synchronous:

React
  |
  v
REST API
  |
  v
Backend


Asynchronous:

Backend
  |
  v
Kafka
  |
  +── Consumer
  +── Consumer
  +── Consumer
```

---

# ADR-012 — JWT Authentication

## Status

Accepted

## Context

The frontend needs authenticated access to protected backend resources.

## Decision

Use JWT-based authentication for the initial implementation.

The access token will be sent using:

```http
Authorization: Bearer <token>
```

## Consequences

Advantages:

* Stateless authentication
* Suitable for REST APIs
* Easy integration with React

Additional token management and security mechanisms may be introduced later.
