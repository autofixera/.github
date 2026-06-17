# Welcome to AutoFixera

> **AutoFixera** is a next-generation, cloud-native auto repair franchise management system.

We simulate a large-scale, nationwide chain of automotive workshops operating across Indonesia. Our platform handles thousands of concurrent transactions—from tracking customer vehicles and parts inventory to issuing complex invoices and streaming real-time revenue analytics back to the central headquarters.

---

## Business Overview

In the fast-paced auto repair industry, tracking parts, labor, and customer histories across hundreds of franchised garages is a logistical nightmare. AutoFixera solves this by decentralizing operations into highly scalable microservices while centralizing the data flow for real-time visibility.

* **Scale:** Built to simulate high-throughput, concurrent workshop transactions.
* **Currency:** Fully localized to Indonesian Rupiah (IDR).
* **Domain Focus:** Automotive repair workflows, servicing schedules, parts replacement, and customer loyalty retention.

---

## Ecosystem Architecture

AutoFixera is built on a modern, event-driven microservices architecture utilizing **Java, Spring Boot 3, Apache Kafka, and PostgreSQL**. 

The system is organized into distinct layers to ensure separation of concerns, scalability, and resilience.

```mermaid
flowchart TB
    %% --- LAYER 1: Presentation ---
    subgraph Presentation["📱 Presentation Layer"]
        direction LR
        Browser([Workshop Manager])
        UI[Analytics Dashboard<br/>Vanilla JS / Glassmorphism]
    end

    %% --- LAYER 2: API Gateway ---
    subgraph Edge["🌐 Edge Layer"]
        Gateway{API Gateway<br/>Spring Cloud Gateway}
    end

    %% --- LAYER 3: Microservices ---
    subgraph Microservices["⚙️ Microservices Layer"]
        direction LR
        Customer[Customer Service<br/>gRPC Server]
        Billing[Billing Service<br/>gRPC Client]
        Analytics[Analytics Service<br/>Event Consumer]
    end

    %% --- LAYER 4: Data & Events ---
    subgraph DataLayer["💾 Data & Event Layer"]
        direction LR
        Kafka{{Apache Kafka<br/>Event Bus}}
        DB_C[(Customer DB)]
        DB_B[(Billing DB)]
        DB_A[(Analytics DB)]
    end

    %% --- CONNECTIONS ---
    
    %% Client to Edge
    Browser -->|REST HTTP| Gateway
    UI -->|Fetch Metrics| Gateway
    
    %% Edge to Services
    Gateway -->|Routes| Customer
    Gateway -->|Routes| Billing
    Gateway -->|Routes| Analytics

    %% Inter-service Sync
    Billing -.->|gRPC Validate| Customer

    %% Event Streaming (Async)
    Customer -.->|Produces Event| Kafka
    Billing -.->|Produces Event| Kafka
    Kafka ===>|Consumes Stream| Analytics

    %% Database Connections
    Customer --- DB_C
    Billing --- DB_B
    Analytics --- DB_A

    %% --- STYLING ---
    classDef layer fill:none,stroke:#94a3b8,stroke-width:2px,stroke-dasharray: 5 5;
    class Presentation,Edge,Microservices,DataLayer layer;
    
    classDef ui fill:#8b5cf6,color:#fff,stroke:#7c3aed,stroke-width:2px;
    classDef gateway fill:#f59e0b,color:#fff,stroke:#d97706,stroke-width:2px;
    classDef service fill:#3b82f6,color:#fff,stroke:#2563eb,stroke-width:2px;
    classDef infra fill:#10b981,color:#fff,stroke:#059669,stroke-width:2px;

    class Browser,UI ui;
    class Gateway gateway;
    class Customer,Billing,Analytics service;
    class Kafka,DB_C,DB_B,DB_A infra;

```

---

## Core Domain Workflow: The Invoice Lifecycle

The core of the AutoFixera business is the workshop service invoice.

When a customer brings their car in for an oil change or major repair, an invoice is generated. This invoice travels through several strictly validated states, ultimately triggering asynchronous Kafka events that update our company-wide analytics dashboard in real-time.

```mermaid
stateDiagram-v2
    direction LR
    
    [*] --> DRAFT : Create Ticket
    DRAFT --> ISSUED : Add Parts & Labor\nApprove Quote
    ISSUED --> PENDING_PAYMENT : Service Completed
    PENDING_PAYMENT --> PAID : Customer Pays (IDR)
    PAID --> [*] : Transaction Closed
    
    note right of PAID
        ⚡ Kafka Event Emitted!
        Analytics Service updates
        HQ Revenue Dashboards.
    end note

```

---

## Repository Index

Our ecosystem is intentionally split into domain-specific repositories. This mono-repo structure ensures strict isolation, independent deployment cycles, and clear domain boundaries.

| Repository | Primary Tech Stack | Description |
| --- | --- | --- |
| **`autofixera-infra`** | Docker, GNU Make, Bash | The heart of our local and cloud orchestration. Contains Docker Compose files and Makefiles to spin up the entire ecosystem seamlessly. |
| **`api-gateway`** | Spring Cloud Gateway | The unified entry point. Handles all routing, CORS, and cross-cutting concerns for incoming HTTP requests. |
| **`customer-service`** | Spring Boot 3, PostgreSQL, gRPC | Manages customer profiles, vehicle data, and exposes high-performance gRPC endpoints for synchronous data validation. |
| **`billing-service`** | Spring Boot 3, PostgreSQL, Kafka | Handles invoice generation, payment processing, and reliably pushes financial events to the message broker. |
| **`analytics-service`** | Spring Boot 3, PostgreSQL, Kafka | A high-performance consumer that materializes Kafka event streams into highly queryable metrics for HQ. |
| **`analytics-ui`** | Vanilla JS/CSS, HTML5, Nginx | A blazing-fast, frontend dashboard featuring a premium dark-mode glassmorphism design. |

