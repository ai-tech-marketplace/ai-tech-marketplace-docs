# System Architecture

## 1. Overview

AI Tech Marketplace uses a multi-repository architecture.

The system is divided into five repositories:

```text
AI Tech Marketplace
│
├── Backend
├── Frontend
├── AI Service
├── Infrastructure
└── Documentation
```

Each repository has a clearly defined responsibility and can be developed and deployed independently.

## 2. Repository Responsibilities

| Repository                     | Responsibility                    | Main Technology             |
| ------------------------------ | --------------------------------- | --------------------------- |
| `ai-tech-marketplace-backend`  | Core business logic and REST APIs | Java, Spring Boot           |
| `ai-tech-marketplace-frontend` | Web application                   | React, TypeScript           |
| `ai-tech-marketplace-ai`       | AI and recommendation services    | Python, FastAPI             |
| `ai-tech-marketplace-infra`    | Infrastructure and deployment     | Docker, Redis, Kafka, Nginx |
| `ai-tech-marketplace-docs`     | Documentation and system design   | Markdown, Mermaid           |

## 3. High-Level Architecture

```text
                         ┌──────────────────────┐
                         │       Customer       │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │   React Frontend     │
                         │  ai-tech-marketplace │
                         │      -frontend       │
                         └──────────┬───────────┘
                                    │
                              HTTPS / REST
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │    Spring Boot       │
                         │       Backend        │
                         └───────┬───────┬──────┘
                                 │       │
                  ┌──────────────┘       └──────────────┐
                  │                                     │
                  ▼                                     ▼
         ┌─────────────────┐                    ┌─────────────┐
         │   PostgreSQL    │                    │    Redis    │
         │ Transactional DB│                    │ Cache/Lock  │
         └────────┬────────┘                    └─────────────┘
                  │
                  ▼
             ┌─────────┐
             │  Kafka  │
             │ Events  │
             └────┬────┘
                  │
        ┌─────────┼─────────┐
        │         │         │
        ▼         ▼         ▼
   Inventory  Notification Analytics


                         Spring Boot Backend
                                │
                                │ Internal HTTP API
                                ▼
                         ┌───────────────┐
                         │  Python AI    │
                         │    Service    │
                         │    FastAPI    │
                         └───────┬───────┘
                                 │
                  ┌──────────────┼──────────────┐
                  │              │              │
                  ▼              ▼              ▼
                RAG        Embedding          LLM
```

## 4. Frontend

The frontend is implemented using React and TypeScript.

Responsibilities include:

* Rendering user interfaces
* Managing client-side state
* Authentication UI
* Product browsing
* Search and filtering
* Shopping cart
* Checkout
* Order management
* AI chat interface
* Admin dashboard

The frontend communicates with the backend through REST APIs.

The frontend must not directly access:

* PostgreSQL
* Redis
* Kafka
* AI databases
* Internal infrastructure services

Communication:

```text
React
  |
  | HTTPS
  v
Spring Boot API
```

## 5. Backend

The backend is the core application service.

Technology:

```text
Java
Spring Boot
Spring Security
Spring Data JPA
PostgreSQL
Redis
Kafka
```

Responsibilities:

* Authentication
* Authorization
* User management
* Product management
* Category management
* Brand management
* Cart management
* Order management
* Inventory management
* Business validation
* Transaction management
* REST APIs
* Communication with AI service
* Publishing domain events

The backend is the authoritative source for transactional business operations.

## 6. AI Service

The AI service is implemented separately using Python and FastAPI.

Responsibilities:

* AI Shopping Assistant
* Natural language product search
* Product recommendation
* Product comparison
* Semantic search
* Embedding generation
* RAG
* LLM integration

The AI service should not directly modify transactional marketplace data.

For example:

```text
AI Service
    |
    | Read/search product information
    v
Backend
```

The backend remains responsible for:

* Creating orders
* Updating inventory
* Authentication
* Authorization
* Transactional operations

## 7. PostgreSQL

PostgreSQL is the primary transactional database.

It stores:

* Users
* Roles
* Products
* Categories
* Brands
* Inventory
* Carts
* Cart items
* Orders
* Order items
* Addresses

PostgreSQL is the source of truth for transactional data.

## 8. Redis

Redis is used for data that benefits from low-latency access.

Initial use cases:

### Cache

Examples:

* Popular products
* Product details
* Categories
* Frequently accessed queries

### Distributed Lock

Redis may be used for concurrency control in operations such as inventory reservation where appropriate.

### Session / Temporary Data

Redis may be used for temporary authentication or application data where required.

Redis must not become the primary source of truth for transactional data.

## 9. Kafka

Kafka is used for asynchronous event-driven communication.

Examples of events:

```text
OrderCreated
OrderConfirmed
OrderCancelled
InventoryReserved
InventoryReleased
ProductUpdated
UserRegistered
```

Example:

```text
Order Service
     |
     | OrderCreated
     v
   Kafka
     |
     +──────────────┬──────────────┐
     ▼              ▼              ▼
Inventory       Notification    Analytics
```

Kafka allows consumers to process events independently.

## 10. AI Communication

The frontend does not directly communicate with the AI service.

The preferred flow is:

```text
Customer
   |
   v
React
   |
   v
Backend
   |
   | Internal API
   v
AI Service
   |
   +── Product Search
   +── Recommendation
   +── RAG
   +── LLM
   |
   v
Backend
   |
   v
React
```

This architecture keeps authentication, authorization, and business rules centralized in the backend.

## 11. Authentication Flow

The system uses JWT-based authentication.

```text
Customer
    |
    | Login
    v
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
    v
Backend
    |
    | JWT
    v
React
```

For protected requests:

```text
React
   |
   | Authorization: Bearer <JWT>
   v
Backend
   |
   | Validate JWT
   v
Controller
```

## 12. Order Flow

A simplified order flow:

```text
Customer
   |
   v
React
   |
   v
Backend
   |
   +── Validate cart
   |
   +── Validate inventory
   |
   +── Calculate order total
   |
   +── Create order
   |
   +── Reserve inventory
   |
   v
PostgreSQL
   |
   v
Publish OrderCreated
   |
   v
Kafka
```

## 13. AI Shopping Flow

Example request:

```text
"I need a laptop around 30 million VND
for Java development and Docker."
```

Flow:

```text
Customer
   |
   v
React Chat
   |
   v
Backend
   |
   v
AI Service
   |
   +── Intent Analysis
   |
   +── Requirement Extraction
   |
   +── Semantic Search
   |
   +── Product Retrieval
   |
   +── Recommendation
   |
   +── LLM Response Generation
   |
   v
Backend
   |
   v
React
```

## 14. Error Handling

The system should isolate failures between services.

Example:

```text
AI Service unavailable
        |
        v
Backend detects failure
        |
        v
Return graceful response
        |
        v
Marketplace remains usable
```

AI functionality should not prevent customers from browsing products or managing existing orders.

## 15. Scalability

The architecture should allow individual services to scale independently.

Example:

```text
                    Load Balancer
                         |
              ┌──────────┼──────────┐
              ▼          ▼          ▼
          Backend 1  Backend 2  Backend 3
              |
              +──── PostgreSQL
              +──── Redis
              +──── Kafka

                    AI Service
                         |
              ┌──────────┼──────────┐
              ▼          ▼          ▼
            AI 1       AI 2       AI 3
```

## 16. Deployment

Each repository can have its own CI/CD pipeline.

```text
GitHub Repository
       |
       v
GitHub Actions
       |
       v
Build
       |
       v
Test
       |
       v
Docker Image
       |
       v
Deployment
```

Repositories are deployed independently:

```text
Backend Repository
        |
        v
Backend Container

Frontend Repository
        |
        v
Frontend Container

AI Repository
        |
        v
AI Container

Infrastructure Repository
        |
        v
Infrastructure Services
```

## 17. Architecture Principles

The project follows these principles:

### Separation of Concerns

Each repository has a clearly defined responsibility.

### Backend as Business Authority

Transactional business rules are enforced by the backend.

### API-Based Communication

Services communicate through defined APIs and events.

### Asynchronous Processing

Kafka is used for operations that do not require synchronous processing.

### Cache Aside

Redis is treated primarily as a cache and temporary data store.

### AI Isolation

AI functionality is isolated from core transactional operations.

### Independent Deployment

Each major service can be built and deployed independently.

### Security by Default

Sensitive data and credentials must not be committed to Git repositories.
