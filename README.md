

---

# EagleEye IoT Platform: Technical Architecture Documentation

**Project Name:** EagleEye Platform  
**Version:** 2.0  
**Architecture Style:** Event-Driven Microservices on Kubernetes  

---

## 1. Executive Summary
EagleEye is a cloud-native, scalable IoT solution designed to ingest, process, and visualize real-time sensor telemetry. The platform leverages a **Microservices Architecture** deployed on **Kubernetes**, utilizing **Kafka** for asynchronous data buffering and high-throughput processing.

The system is designed to handle:
*   High-velocity data ingestion from distributed IoT devices.
*   Real-time alerting based on configurable business rules.
*   Complex analytics using ClickHouse (OLAP) and PostgreSQL (OLTP).
*   Seamless synchronization with enterprise ERP systems (1C).

---

## 2. High-Level Architecture
The system is divided into four logical domains: **Ingestion**, **Core Services**, **Alerting**, and **Infrastructure**.

### 2.1 Architectural Diagram
*(Refer to the system topology diagram visualizing the flow from Sensors $\to$ Kafka $\to$ Workers $\to$ Databases)*.

---

## 3. Microservices Breakdown

### 3.1 Domain: Sensor Ingestion
Responsible for the entry point of raw telemetry data into the cluster.

*   **Service:** `Sensor Messaging API`
    *   **Role:** High-performance Gateway.
    *   **Protocol:** REST / gRPC.
    *   **Responsibility:** Authenticates devices, validates JSON payloads, and immediately produces messages to the **Kafka Sensor Topic**. It does *not* write to the database directly to ensure low latency.
*   **Service:** `Sensor Messaging Worker`
    *   **Role:** Background Consumer.
    *   **Responsibility:** Subscribes to the **Kafka Sensor Topic**. It processes raw messages, performs data normalization, and persists data to:
        1.  **ClickHouse:** For historical time-series analytics.
        2.  **PostgreSQL:** For current device state (last known location/status).

### 3.2 Domain: Core APIs
The backbone of the application, handling user interactions and data aggregation.

*   **Service:** `Main API` (`mymainapi`)
    *   **Role:** The primary backend for the Frontend UI.
    *   **Responsibility:** Manages User Authentication, Asset Management, and Organization settings.
    *   **Data Access:** Reads/Writes user data to **PostgreSQL**.
    *   **Inter-service:** Provides gRPC endpoints for other services (e.g., serving email templates to the Notification Worker).
*   **Service:** `Measurement API`
    *   **Role:** Analytics Aggregator.
    *   **Responsibility:** Calculates complex metrics (averages, min/max) from **ClickHouse**.
    *   **Consumers:** Used by the `Alerting Worker` to check rule violations and by the `Public API` for external consumers.
*   **Service:** `Public API`
    *   **Role:** External Gateway.
    *   **Responsibility:** Exposes a limited, secure set of data to third-party developers or partners, proxying requests to the internal Core APIs.

### 3.3 Domain: Alerting & Notifications
An event-driven subsystem for detecting anomalies and notifying users.

*   **Service:** `Alerting Worker`
    *   **Role:** Rule Engine.
    *   **Logic:** Periodically fetches aggregated data from `Measurement API`. Compares data against user-defined thresholds (e.g., "Temperature > 50°C").
    *   **Action:** If a rule is breached, it pushes an event to the **Kafka Notification Queue**.
*   **Service:** `Notification Worker`
    *   **Role:** Delivery Agent.
    *   **Logic:** Consumes the Notification Queue.
        1.  Fetches HTML templates from `Main API` (via gRPC).
        2.  Delivers messages via **Email** (SMTP) or **Telegram** API.
        3.  Logs the alert history to **PostgreSQL**.

### 3.4 Domain: Enterprise Integration
*   **Service:** `Integration OneC API` (`integrationonecapi`)
    *   **Role:** ERP Bridge.
    *   **Responsibility:** Accepts webhook calls from the external **1C ERP System**. Syncs contractors, employees, and vehicle assets into the EagleEye **PostgreSQL** database.

---

## 4. Data Persistence & Messaging Infrastructure

### 4.1 Apache Kafka (The Nervous System)
Kafka acts as the buffer to decouple services and ensure data is never lost during traffic spikes.
*   **Topic A: `sensor-data`**: High-throughput raw telemetry. Retention policy set for short-term replayability.
*   **Topic B: `notification-queue`**: Low-throughput, high-priority alert events.

### 4.2 ClickHouse (Time-Series Storage)
*   **Usage:** OLAP (Online Analytical Processing).
*   **Why:** Optimized for writing millions of sensor readings per minute and running fast aggregation queries (SUM, AVG) over large datasets.
*   **Data:** Temperature, Humidity, GPS coordinates, Battery levels, RPM.

### 4.3 PostgreSQL (Relational Storage)
*   **Usage:** OLTP (Online Transaction Processing).
*   **Why:** Ensures ACID compliance for critical business data.
*   **Data:** User Profiles, Auth Tokens, Device Metadata, Alert Rules, Notification Logs.

---

## 5. Deployment Strategy (DevOps)

The project utilizes **Helm** for packaging and **GitOps** for deployment.

### 5.1 Helm Chart Structure
The `apps/` directory functions as a Monorepo, containing individual charts for each microservice.
*   **Templates:** Standardized Kubernetes manifests (`Deployment`, `Service`, `Ingress`, `ConfigMap`).
*   **Values:** Environment-specific configurations (Production vs. Staging).

### 5.2 Kubernetes Components
*   **Ingress Controllers:** Manage external access (HTTP/HTTPS) to `Main API`, `Sensor Messaging API`, and `Integration API`.
*   **Secrets Management:** Sensitive credentials (DB passwords, Telegram Tokens) are injected via Kubernetes Secrets (managed by `infrastructure/commands_secret`).
*   **Scaling:** Stateless services (`Sensor API`, `Main API`) are configured with **Horizontal Pod Autoscalers (HPA)** to handle load variations.

---

## 6. Communication Protocols
*   **External Traffic:** HTTPS (JSON REST).
*   **Internal Service-to-Service:** gRPC (Protobuf) for high-performance, strongly typed communication (e.g., `Notification Worker` $\leftrightarrow$ `Main API`).
*   **Async Communication:** Kafka Binary Protocol.

---

## 7. Security Considerations
1.  **Ingress Layer:** SSL/TLS termination at the load balancer.
2.  **Service Isolation:** Network Policies restrict traffic so only specific workers can access the Database ports.
3.  **Authentication:** `Main API` issues JWT tokens for user sessions; `Sensor API` uses API Keys for device validation.




![Architecture Diagram](images/architecture.png)

