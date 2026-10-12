# System Context Diagram (C4 Model Level 1)

## 1. Overview

The System Context diagram provides a high-level view of the **AI Tech Marketplace** system, showing its boundaries, the human actors who interact with it, and external systems it integrates with.

---

## 2. Context Diagram

```mermaid
flowchart TD
    %% Styling
    classDef person fill:#08427b,stroke:#073b6f,color:#ffffff,stroke-width:2px;
    classDef system fill:#1168bd,stroke:#0e5292,color:#ffffff,stroke-width:2px;
    classDef external fill:#999999,stroke:#666666,color:#ffffff,stroke-width:2px;

    %% Actors
    customer["fa:fa-user Customer<br/>[Person]<br/>Discovers, compares, purchases tech products, and chats with the AI Shopping Assistant"]:::person
    seller["fa:fa-user-tie Seller<br/>[Person]<br/>Manages product listings, pricing, and monitors product sales"]:::person
    admin["fa:fa-user-shield Administrator<br/>[Person]<br/>Manages system users, catalog categories, brands, orders, and system analytics"]:::person

    %% Core System
    systemBoundary["AI Tech Marketplace Platform<br/>[Software System]<br/>Provides technology e-commerce, shopping cart, order management, inventory tracking, and AI-powered recommendations"]:::system

    %% External Systems
    llmProvider["fa:fa-brain External LLM Provider<br/>[External System]<br/>Provides large language model inference for semantic understanding and response generation"]:::external
    paymentGateway["fa:fa-credit-card Payment Gateway<br/>[External System]<br/>Processes customer online payments (e.g., VNPay, MoMo, Stripe)"]:::external
    emailService["fa:fa-envelope Notification / Email Service<br/>[External System]<br/>Sends transactional emails for registration, order confirmation, and updates"]:::external
    objectStorage["fa:fa-cloud Object Storage<br/>[External System]<br/>Stores and serves product images, specifications sheets, and media assets"]:::external

    %% Relationships - Actors to System
    customer -->|"Browses catalog, manages cart, places orders, chats with AI Assistant [HTTPS/REST]"| systemBoundary
    seller -->|"Manages products and inventory, reviews order items [HTTPS/REST]"| systemBoundary
    admin -->|"Governs users, catalog, orders, and platform health [HTTPS/REST]"| systemBoundary

    %% Relationships - System to External
    systemBoundary -->|"Sends prompt context, retrieves embeddings and completions [HTTPS/REST]"| llmProvider
    systemBoundary -->|"Initiates payments, receives webhook callbacks [HTTPS/REST]"| paymentGateway
    systemBoundary -->|"Dispatches transactional notifications [SMTP / HTTPS]"| emailService
    systemBoundary -->|"Uploads and retrieves product media [HTTPS / S3 API]"| objectStorage
```

---

## 3. Actors

| Actor | Type | Description |
| --- | --- | --- |
| **Customer** | Person | End-user shopping for laptops, components, accessories, comparing products, and consulting the AI Shopping Assistant. |
| **Seller** | Person | Merchant managing their product catalog, pricing, and stock levels. |
| **Administrator** | Person | Internal platform operator governing accounts, categories, brands, orders, and reviewing analytics. |

---

## 4. Systems

| System | Classification | Description |
| --- | --- | --- |
| **AI Tech Marketplace** | Core System | The central platform encompassing React frontend, Spring Boot backend, Python AI service, PostgreSQL database, Redis cache, and Kafka messaging. |
| **External LLM Provider** | External System | Third-party AI model API (e.g., Gemini, OpenAI) utilized by the AI service for embeddings, RAG reasoning, and natural language chat. |
| **Payment Gateway** | External System | External payment service provider processing financial transactions and issuing webhook notifications. |
| **Notification / Email Service** | External System | Third-party transactional delivery service (e.g., SendGrid, Mailgun) handling automated notification dispatches. |
| **Object Storage** | External System | Cloud object storage (e.g., AWS S3, MinIO, Cloudinary) storing product gallery images and static attachments. |
