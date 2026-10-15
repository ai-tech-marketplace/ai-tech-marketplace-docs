# AI Tech Marketplace

AI Tech Marketplace is a modern technology e-commerce and multi-vendor marketplace platform featuring an AI-powered Shopping Assistant.

The platform enables customers to discover, compare, purchase, and review technology products—such as laptops, smartphones, PC components, monitors, and smart accessories—with intelligent recommendations powered by LLMs and Retrieval-Augmented Generation (RAG).

---

## Main Goals & Key Architectural Capabilities

The project demonstrates production-grade software engineering practices across distributed microservices and asynchronous systems:

* **Modern Web Development**: React 19 + TypeScript frontend with Tailwind CSS and responsive UI.
* **Robust Core APIs**: Spring Boot 4 RESTful APIs with strict validation and standardized response envelopes (`ApiResponse<T>`).
* **Authentication & RBAC**: Stateless JWT authentication with granular roles (`ROLE_CUSTOMER`, `ROLE_SELLER`, `ROLE_ADMIN`).
* **High-Concurrency Checkout**:
  * Stock reservation protected with database pessimistic write locks (`PESSIMISTIC_WRITE`).
  * Distributed locks using **Redis** (`RedisLockRegistry`) outside transaction boundaries to prevent concurrent double-booking.
* **Event-Driven Architecture**:
  * Asynchronous domain events (`OrderCreatedEvent`, `OrderCancelledEvent`) published to **Apache Kafka** after transaction commit.
  * User-level message ordering guaranteed via `userId` partition key.
  * Resilient Kafka consumers for notifications and analytics with Dead Letter Topics (DLT) and backoff retries.
* **Promotion Engine**: Pre-checkout coupon validation with dynamic shipping fee deduction and automatic quota rollback on order cancellation.
* **Object Storage**: S3-compatible SeaweedFS integration for multi-image product galleries.
* **AI-Powered Recommendation**: Python FastAPI service with vector embeddings, semantic search, and LLM orchestration.
* **Containerized Infrastructure**: Docker Compose setup for PostgreSQL, Redis, Apache Kafka (KRaft), and SeaweedFS S3.

---

## Repositories Ecosystem

The platform is organized into modular repositories:

| Repository | Responsibility | Technology Stack | Current Status |
| :--- | :--- | :--- | :--- |
| [`ai-tech-marketplace-docs`](file:///home/hieunv1/Documents/ai-tech-marketplace/ai-tech-marketplace-docs) | Architecture, requirements, database schemas & API specs | Markdown, Mermaid | **Active / Maintained** |
| [`ai-tech-marketplace-backend`](file:///home/hieunv1/Documents/ai-tech-marketplace/ai-tech-marketplace-backend) | Core transactional business services & REST APIs | Java 21, Spring Boot, PostgreSQL, Redis, Kafka, SeaweedFS S3 | **Phases 2 & 3 Backend Complete** |
| [`ai-tech-marketplace-frontend`](file:///home/hieunv1/Documents/ai-tech-marketplace/ai-tech-marketplace-frontend) | Customer web application & shopping interface | React 19, TypeScript, Tailwind CSS | **Initialized (Next Up)** |
| [`ai-tech-marketplace-ai`](file:///home/hieunv1/Documents/ai-tech-marketplace/ai-tech-marketplace-ai) | AI shopping assistant, vector search & RAG pipeline | Python, FastAPI, pgvector, LLM | **Planned (Phase 4)** |
| [`ai-tech-marketplace-infra`](file:///home/hieunv1/Documents/ai-tech-marketplace/ai-tech-marketplace-infra) | Deployment scripts, container orchestration & CI/CD | Docker Compose, Nginx, Prometheus, Grafana | **Initialized** |

---

## High-Level Architecture

```text
                           +------------------------+
                           |  Customer / Web User   |
                           +-----------+------------+
                                       |
                                       v
                           +------------------------+
                           |     React Frontend     |
                           +-----------+------------+
                                       |
                                       | REST API (/api/v1/*)
                                       v
        +-------------------------------------------------------------+
        |                 Spring Boot 4.x Backend                     |
        |  (Auth, Products, Cart, Addresses, Coupons, Orders, Events) |
        +---+------------------+-------------------+---------------+--+
            |                  |                   |               |
            | JDBC / Flyway    | Lettuce / Locks   | AWS SDK S3    | Kafka Producer
            v                  v                   v               v
     +--------------+   +--------------+   +---------------+   +---------------+
     |  PostgreSQL  |   |    Redis     |   |   SeaweedFS   |   | Apache Kafka  |
     |  (Port 55433)|   | (Distributed |   | (Port 8333)   |   | (Port 9092)   |
     +--------------+   |  Locks)      |   +---------------+   +-------+-------+
                        +--------------+                               |
                                                                       | Topic: order-events
                                                                       v
                                                        +-----------------------------+
                                                        |      Kafka Consumers        |
                                                        |  - Notification Consumer    |
                                                        |  - Analytics Consumer       |
                                                        |  - DLT (Dead Letter Topic)  |
                                                        +-----------------------------+
```

---

## Documentation Structure

* [**Requirements**](requirements/):
  * [Product Requirements](requirements/requirements.md) — Functional and non-functional requirements.
  * [Business Rules](requirements/business-rules.md) — Pricing, cart lifecycle, inventory, coupons, and orders rules.
  * [Use Cases](requirements/use-cases.md) — Customer, seller, and admin workflows.
* [**Architecture**](architecture/):
  * [System Architecture](architecture/system-architecture.md) — High-level architecture, module breakdown, and tech stack.
  * [Architecture Decisions (ADRs)](architecture/architecture-decisions.md) — ADR-001 through ADR-012 documenting key architectural decisions.
  * [Database Design](architecture/database-design.md) — Relational model, indexing strategy, and JSONB specification handling.
  * [API Design Guide](architecture/api-design.md) — REST conventions, envelope format, error structures, and HTTP statuses.
* [**Database**](database/):
  * [Schema](database/schema.md) — Complete table definitions across all entities.
  * [Migrations](database/migrations.md) — Migration plan aligning with Flyway `V1` to `V9`.
  * [Seed Data](database/seed-data.md) — Sample categories, brands, tech products, and specifications.
* [**API Specifications**](api/):
  * [Authentication](api/authentication.md) (`/api/v1/auth`)
  * [Products Catalog](api/products.md) (`/api/v1/products`)
  * [Categories & Brands](api/brands.md) (`/api/v1/categories`, `/api/v1/brands`)
  * [Shopping Cart](api/cart.md) (`/api/v1/cart`)
  * [Shipping Addresses](api/addresses.md) (`/api/v1/addresses`)
  * [Coupons & Promotions](api/coupons.md) (`/api/v1/coupons`, `/api/v1/admin/coupons`)
  * [Commission Settings](api/commission.md) (`/api/v1/admin/commission-settings`)
  * [Financial Analytics](api/analytics.md) (`/api/v1/{admin,seller,customer}/analytics`)
  * [Orders & Checkout](api/orders.md) (`/api/v1/orders`)
  * [Reviews](api/reviews.md) (`/api/v1/reviews`)
  * [AI Assistant](api/ai.md) (`/api/v1/ai`)
* [**Diagrams**](diagrams/):
  * [C4 System Context](diagrams/system-context.md)
  * [C4 Container Diagram](diagrams/container-diagram.md)
  * [Sequence Diagrams](diagrams/sequence-diagrams.md) (Auth, Checkout, AI Assistant)
  * [Entity-Relationship Diagram (ERD)](diagrams/erd.md)
* [**Development**](development/):
  * [Roadmap](development/roadmap.md) — Implementation milestones, deliverables, and progress tracking.

---

## Current Project Status

| Phase | Milestone | Status | Key Deliverables |
| :--- | :--- | :--- | :--- |
| **Phase 1** | System Analysis & Architecture | **Completed** | Full SRS, ADRs (001–012), Database Schema, API Contracts, C4 & Sequence Diagrams. |
| **Phase 2** | Core MVP: Catalog & Shopping Cart | **Backend Complete** | Flyway V1–V6; JWT Auth & RBAC; Categories, Brands & Products APIs; JSONB Specs; S3 Image Uploads; Cart with guest merge & validation. Frontend UI next. |
| **Phase 3** | Orders, Checkout & Event Streaming | **Backend Complete** | Flyway V7–V9; Shipping Addresses; Coupon Engine with dynamic rules; Transactional Checkout; Pessimistic Stock Locking; Redis Distributed Locking; Kafka Publisher (`userId` partition key) & Consumers with DLT. |
| **Phase 4** | AI Assistant & Semantic Search | **Planned** | Python FastAPI service, pgvector, product catalog vectorization, RAG recommendation chat widget. |
| **Phase 5** | Multi-Vendor Operations & Production | **Future** | Seller portal, admin metrics dashboard, CI/CD pipelines, Prometheus/Grafana monitoring, cloud deployment. |

For detailed breakdown and task tracking, please refer to the [Development Roadmap](development/roadmap.md).
