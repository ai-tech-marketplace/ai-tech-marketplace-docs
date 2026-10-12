# Container Diagram (C4 Model Level 2)

## 1. Overview

The Container diagram zooms into the **AI Tech Marketplace Platform**, showing the high-level architecture of its constituent containers (applications, databases, message brokers), their individual responsibilities, runtime technologies, and inter-container protocols.

---

## 2. Container Diagram

```mermaid
flowchart TB
    %% Styling
    classDef person fill:#08427b,stroke:#073b6f,color:#ffffff,stroke-width:2px;
    classDef webapp fill:#1168bd,stroke:#0e5292,color:#ffffff,stroke-width:2px;
    classDef service fill:#1d70b8,stroke:#144e82,color:#ffffff,stroke-width:2px;
    classDef db fill:#23527c,stroke:#173753,color:#ffffff,stroke-width:2px;
    classDef broker fill:#d9534f,stroke:#b52b27,color:#ffffff,stroke-width:2px;
    classDef external fill:#999999,stroke:#666666,color:#ffffff,stroke-width:2px;

    %% Actors
    user["fa:fa-user Customer / Seller / Admin<br/>[Browser User]"]:::person

    subgraph Platform["AI Tech Marketplace Platform [Boundary]"]
        %% Frontend
        frontend["React Web Application<br/>[Container: React, TypeScript]<br/>Single-page application delivering user interfaces for product catalog, shopping cart, checkout, admin, and AI chat"]:::webapp

        %% Backend
        backend["Core Backend Service<br/>[Container: Java, Spring Boot 3]<br/>Transactional business authority, JWT security, REST APIs, domain event publisher"]:::service

        %% AI Service
        aiService["AI Assistant Service<br/>[Container: Python, FastAPI]<br/>Handles natural language intent, product semantic search, RAG retrieval, and LLM orchestration"]:::service

        %% Data Stores
        postgres[("PostgreSQL Database<br/>[Container: PostgreSQL 16]<br/>Stores users, catalog, orders, carts, inventory, and conversations (Source of Truth)")]:::db

        redis[("Redis In-Memory Store<br/>[Container: Redis 7]<br/>Caches hot product queries, sessions, and provides distributed locking")]:::db

        %% Event Streaming
        kafka[["Kafka Event Broker<br/>[Container: Apache Kafka]<br/>Publishes and streams domain events (OrderCreated, InventoryReserved, etc.)"]]:::broker

        %% Background Consumers
        workers["Background Event Listeners<br/>[Container: Spring Boot Workers]<br/>Asynchronously processes notification dispatching, inventory adjustments, and analytics"]]:::service
    end

    %% External Systems
    llmProvider["fa:fa-brain External LLM Provider<br/>[External API: Gemini / OpenAI]"]:::external
    paymentGateway["fa:fa-credit-card Payment Gateway<br/>[External API: VNPay / Stripe]"]:::external
    emailService["fa:fa-envelope Notification Service<br/>[External API: SendGrid / SMTP]"]:::external
    objectStorage["fa:fa-cloud Cloud Object Storage<br/>[External Storage: S3 / MinIO]"]:::external

    %% Flow connections
    user -->|"HTTPS / Web Browser"| frontend
    frontend -->|"REST API / JSON<br/>[HTTPS, Bearer JWT]"| backend

    backend -->|"Reads / Writes Transactional Data<br/>[JDBC / TCP:5432]"| postgres
    backend -->|"Caches queries & acquires locks<br/>[RESP / TCP:6379]"| redis
    backend -->|"Publishes Domain Events<br/>[Kafka Protocol / TCP:9092]"| kafka

    backend -->|"Internal AI Request<br/>[HTTP / REST]"| aiService
    aiService -->|"Grounding Data Retrieval<br/>[Internal HTTP / REST]"| backend
    aiService -->|"Inference & Embeddings<br/>[HTTPS / REST]"| llmProvider

    kafka -->|"Subscribes to Events<br/>[Kafka Consumer Protocol]"| workers
    workers -->|"Persists async updates<br/>[JDBC]"| postgres
    workers -->|"Sends transactional emails<br/>[HTTPS / SMTP]"| emailService

    backend -->|"Processes payment checkout<br/>[HTTPS / REST]"| paymentGateway
    backend -->|"Uploads & serves images<br/>[HTTPS / S3 API]"| objectStorage
    frontend -->|"Loads image assets<br/>[HTTPS / CDN]"| objectStorage
```

---

## 3. Container Breakdown

| Container | Technology Stack | Responsibility | Scaling & Protocol |
| --- | --- | --- | --- |
| **React Web Application** | React, TypeScript, Tailwind CSS | UI presentation, state management, client routing, AI chat widget. | Static hosting (Nginx / Vercel), communicates via HTTPS/REST. |
| **Core Backend Service** | Java 21, Spring Boot 3, Spring Data JPA, Spring Security | Business validation, order checkout transactions, inventory integrity, JWT token issuing, internal routing. | Horizontal container scaling; stateless; communicates with DB via JDBC pool (HikariCP). |
| **AI Assistant Service** | Python 3.11+, FastAPI, LangChain/LlamaIndex | Semantic search, product embeddings generation, prompt engineering, RAG context assembly. | Horizontal container scaling; GPU/CPU worker nodes; internal HTTP. |
| **PostgreSQL Database** | PostgreSQL 16 | Relational transactional persistence, primary keys, foreign keys, JSONB specifications, Flyway migrations. | Primary-Replica architecture, volume-backed persistence. |
| **Redis Store** | Redis 7 | In-memory query caching, rate limit buckets, distributed locking for inventory checkout. | Standalone in dev, Redis Cluster/Sentinel in production. |
| **Kafka Event Broker** | Apache Kafka 3.x, KRaft | Decoupled event-driven streaming for order lifecycle, stock reserve events, and audit logging. | Multi-partition topics, consumer groups. |
| **Background Listeners** | Spring Kafka Listeners | Asynchronous consumers handling email notification triggers, inventory updates, and metrics aggregation. | Autoscaled by consumer lag. |
