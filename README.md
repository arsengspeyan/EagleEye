
# EagleEye


Project Overview 

The EagleEye Platform is a cloud-native, IoT-enabled solution designed to ingest, process, and visualize real-time sensor data alongside operational business information. It provides a scalable, integrated system to manage distributed sensor networks, synchronize with enterprise systems, and deliver actionable insights through a user-friendly interface. Built with a microservices architecture, it leverages modern DevOps practices and GitOps workflows for continuous deployment.

🔹 Core Components

### Backend APIs (Microservices):

🔹 EagleEye.Main.API:

 The primary REST and gRPC API, providing endpoints for authentication, monitoring, asset management (e.g., contractors, vehicles, warehouses), and tracker data. It serves as the backbone for internal and external client interactions.


🔹 EagleEye.Integration.OneC.API:

 A specialized microservice integrating with the 1C ERP system, handling data synchronization for entities such as contractors, organizations, employees, and financial records.


🔹 EagleEye.Calculator.API:

 A lightweight service focused on processing sensor telemetry and performing calculations for downstream analytics. Each API follows a layered design with a Business Logic Layer (BLL) for domain rules and validation, and a Data Access Layer (DAL) using Entity Framework Core for PostgreSQL persistence.



### Data & Storage Layer:

#### PostgreSQL: 

Manages transactional data, including user profiles, configurations, and organizational metadata.

#### ClickHouse:

Handles high-volume sensor data analytics with optimized time-series storage.

#### Kafka:

Facilitates event-driven data pipelines for sensor ingestion, notification queues, and real-time processing.

#### Alerting & Notification System:

Monitors sensor and business data, triggering alerts based on predefined rules. Delivers notifications via Email, Telegram, and Kafka queues, ensuring scalability and reliability.

#### Web Tier (Angular Frontend):

A modern, responsive Angular-based interface that visualizes sensor data, manages organizational assets, and configures alerting rules. It communicates with backend APIs via REST and gRPC, providing an intuitive experience for end-users.

🔹 Key Capabilities

Real-time sensor data ingestion and processing from distributed devices.
Seamless integration with the 1C ERP system for business data synchronization.
Centralized API ecosystem for scalable client access.
Event-driven architecture with Kafka for real-time data flow.
Advanced analytics powered by ClickHouse and Calculator.API.
Multi-channel alerting system (Email, Telegram) with Kafka support.
Interactive Angular frontend for data visualization and management.

This platform demonstrates a robust, scalable architecture suitable for IoT and enterprise applications, deployed using Kubernetes with ArgoCD for GitOps-driven updates.
