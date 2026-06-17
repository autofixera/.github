# Welcome to AutoFixera 👋

AutoFixera is a next-generation, cloud-native auto repair franchise management system. 

We simulate a large-scale, nationwide chain of automotive workshops operating across Indonesia. Our platform handles thousands of concurrent transactions—from tracking customer vehicles and parts inventory to issuing complex invoices and streaming real-time revenue analytics back to the central headquarters.

---

## 🏢 Business Overview

In the fast-paced auto repair industry, tracking parts, labor, and customer histories across hundreds of franchised garages is a logistical nightmare. AutoFixera solves this by decentralizing operations into highly scalable microservices while centralizing the data flow for real-time visibility.

- **Scale:** Simulates high-throughput workshop transactions.
- **Currency:** Fully localized to Indonesian Rupiah (IDR).
- **Domain:** Automotive repair, servicing, parts replacement, and customer loyalty.

---

## 🏗️ Ecosystem Architecture

AutoFixera is built on a modern event-driven microservices architecture utilizing **Java, Spring Boot 3, Apache Kafka, and PostgreSQL**.

```mermaid
graph TD
    %% Clients
    Browser([Workshop Manager Browser])
    
    %% Gateway
    Gateway[API Gateway<br/>Spring Cloud Gateway]
    Browser -->|HTTP REST| Gateway
    
    %% UI
    UI[Analytics UI<br/>Vanilla JS / Glassmorphism]
    Browser -->|Fetches Dashboard| UI
    
    %% Core Services
    subgraph Microservices
        Customer[Customer Service<br/>Manages Profiles & Vehicles]
        Billing[Billing Service<br/>Manages Invoices & Payments]
        Analytics[Analytics Service<br/>Real-time Aggregation]
    end
    
    Gateway -->|Routes Traffic| Customer
    Gateway -->|Routes Traffic| Billing
    UI -->|Fetches Data| Analytics
    
    %% Inter-service
    Billing -.->|gRPC: Validate Customer| Customer
    
    %% Event Streaming
    Kafka{{Apache Kafka<br/>Event Bus}}
    Customer -.->|Produces: CustomerEvent| Kafka
    Billing -.->|Produces: BillingEvent| Kafka
    Kafka ===>|Consumes Stream| Analytics
    
    %% Databases
    DB_C[(Customer DB)]
    DB_B[(Billing DB)]
    DB_A[(Analytics DB)]
    
    Customer --- DB_C
    Billing --- DB_B
    Analytics --- DB_A

    classDef service fill:#3b82f6,color:#fff,stroke:#2563eb;
    classDef infra fill:#10b981,color:#fff,stroke:#059669;
    classDef ui fill:#8b5cf6,color:#fff,stroke:#7c3aed;
    classDef gateway fill:#f59e0b,color:#fff,stroke:#d97706;
    
    class Customer,Billing,Analytics service;
    class DB_C,DB_B,DB_A,Kafka infra;
    class UI ui;
    class Gateway gateway;
```

---

## 💸 Core Domain Workflow: The Invoice Lifecycle

The core of the AutoFixera business is the workshop service invoice. 

When a customer brings their car in for an oil change or major repair, an invoice is generated. This invoice travels through several states, triggering asynchronous Kafka events that update our company-wide analytics dashboard in real-time.

```mermaid
stateDiagram-v2
    [*] --> DRAFT: Mechanic creates ticket
    
    DRAFT --> ISSUED: Customer approves quote
    note right of DRAFT
      Adding parts (e.g., Brake Pads)
      and labor (e.g., Oil Change)
    end note
    
    ISSUED --> PENDING_PAYMENT: Service completed
    note right of ISSUED
      Waiting on mechanics 
      to finish the job.
    end note
    
    PENDING_PAYMENT --> PAID: Customer pays bill (IDR)
    note right of PENDING_PAYMENT
      Cashier accepts payment.
    end note
    
    PAID --> [*]: Transaction complete
    
    note left of PAID
      ⚡ Kafka Event Emitted!
      Analytics Service updates
      HQ Revenue Dashboards.
    end note
```

---

## 📂 Repository Index

Our ecosystem is split into domain-specific repositories to ensure strict isolation and independent deployment cycles.

| Repository | Description | Primary Tech Stack |
| --- | --- | --- |
| **`autofixera-infra`** | The heart of our local and cloud orchestration. Contains Docker Compose files and Makefiles to spin up the entire ecosystem. | Docker, GNU Make, Bash |
| **`api-gateway`** | The unified entry point. Handles all routing and cross-cutting concerns for incoming HTTP requests. | Spring Cloud Gateway |
| **`customer-service`** | Manages customer profiles, vehicle data, and exposes gRPC endpoints for synchronous validation. | Spring Boot 3, PostgreSQL, gRPC |
| **`billing-service`** | Handles invoice generation, payment processing, and pushes financial events to Kafka. | Spring Boot 3, PostgreSQL, Kafka |
| **`analytics-service`** | A high-performance consumer that materializes Kafka event streams into queryable metrics for HQ. | Spring Boot 3, PostgreSQL, Kafka |
| **`analytics-ui`** | A blazing-fast, 100/100 Lighthouse score frontend dashboard with a premium dark-mode glassmorphism design. | Vanilla JS/CSS, HTML5, Nginx |
