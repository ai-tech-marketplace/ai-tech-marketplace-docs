# AI Tech Marketplace

AI Tech Marketplace is a technology e-commerce and marketplace platform with an AI-powered Shopping Assistant.

The system allows customers to discover, compare, purchase, and manage technology products such as laptops, smartphones, PC components, monitors, accessories, and other electronic devices.

## Main Goals

The project is designed to demonstrate:

* Modern web application development
* RESTful API design
* Authentication and authorization
* Transactional business logic
* Distributed system concepts
* Redis caching
* Kafka event-driven communication
* AI-powered product recommendation
* RAG and semantic search
* LLM integration
* Docker-based infrastructure
* CI/CD
* Monitoring and logging

## Repositories

The project is divided into multiple repositories.

| Repository                     | Responsibility       | Technology                  |
| ------------------------------ | -------------------- | --------------------------- |
| `ai-tech-marketplace-backend`  | Core business APIs   | Java, Spring Boot           |
| `ai-tech-marketplace-frontend` | Web application      | React, TypeScript           |
| `ai-tech-marketplace-ai`       | AI services          | Python, FastAPI             |
| `ai-tech-marketplace-infra`    | Infrastructure       | Docker, Redis, Kafka, Nginx |
| `ai-tech-marketplace-docs`     | System documentation | Markdown, Mermaid           |

## High-Level Architecture

Customer
   |
   v
React Frontend
   |
   | REST API
   v
Spring Boot Backend
   |
   +----------------+
   |                |
   v                v
PostgreSQL        Redis
   |
   v
Kafka
   |
   +----------+-----------+
   |          |           |
   v          v           v
Inventory  Notification  Analytics

Spring Boot Backend
        |
        | Internal API
        v
Python AI Service
        |
        +--------+--------+
        |        |        |
        v        v        v
       RAG   Embedding    LLM

## Documentation Structure

### Requirements

Contains product requirements, use cases, and business rules.

### Architecture

Contains system architecture, database design, API design, and architecture decisions.

### API

Contains detailed API contracts between services.

### Diagrams

Contains system and database diagrams.

### Development

Contains the project roadmap and development process.

## Project Status

Current phase:

* **Phase 1 — System Analysis & Architecture**: Completed / In Review
* **Phase 2 — Core MVP: Catalog & Shopping Cart**: In Progress (Spring Boot backend initialization complete; catalog and cart implementation underway)

For the complete breakdown of project milestones and multi-repository deliverables, see the [Development Roadmap](development/roadmap.md).
