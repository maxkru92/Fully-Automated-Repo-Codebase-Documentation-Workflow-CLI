<img width="2816" height="1536" alt="Gemini_Generated_Image_b7bxlib7bxlib7bx" src="https://github.com/user-attachments/assets/3def3cf5-6829-44d7-8bbf-dcb1f90148b8" />

# Fully-Automated-Repo-Codebase-Documentation-Workflow-CLI
## CLI Light & CLI Enterprise
Act as a senior software architect. Implement the following system design:

# System Architecture

The following system design document outlines a production-ready architecture for the "Automated Quant-Finance Codebase Documentation & Quality Workflow CLI" project. It is designed to be scalable, maintainable, and secure for startups handling 100s to 10K+ users, with a growth target of 20k-100k daily active users and 1k-5k peak concurrent users (150-800 peak RPS, with 3x-5x burst traffic). The design adheres to a target monthly cost of $500-2000.

---

# System Design Document: Automated Quant-Finance Codebase Documentation & Quality Workflow CLI

## Table of Contents

1.  [Executive Summary](#1-executive-summary)
2.  [System Ideatosystem](#2-system-ideatosystem)
    *   [High-Level Architecture Diagram](#high-level-architecture-diagram)
3.  [Component Design](#3-component-design)
    *   [CLI Application](#cli-application)
    *   [Load Balancer / API Gateway](#load-balancer--api-gateway)
    *   [Backend API Service](#backend-api-service)
    *   [Worker Service](#worker-service)
    *   [Authentication Service](#authentication-service)
    *   [Database (PostgreSQL)](#database-postgresql)
    *   [Cache (Redis)](#cache-redis)
    *   [Message Queue (SQS)](#message-queue-sqs)
    *   [Object Storage (S3)](#object-storage-s3)
4.  [Data Model](#4-data-model)
    *   [Entities & Relationships](#entities--relationships)
    *   [Indexing Strategy](#indexing-strategy)
    *   [Connection Pooling](#connection-pooling)
    *   [Caching Layer Strategy](#caching-layer-strategy)
5.  [API Design](#5-api-design)
    *   [Authentication](#authentication)
    *   [Key Endpoints & Contracts](#key-endpoints--contracts)
6.  [Scalability Strategy](#6-scalability-strategy)
    *   [Horizontal Scaling](#horizontal-scaling)
    *   [Database Scaling](#database-scaling)
    *   [Caching](#caching)
    *   [Asynchronous Processing](#asynchronous-processing)
7.  [Security Considerations](#7-security-considerations)
    *   [Authentication & Authorization](#authentication--authorization)
    *   [Rate Limiting](#rate-limiting)
    *   [Input Validation](#input-validation)
    *   [Secrets Management](#secrets-management)
    *   [Security Headers](#security-headers)
    *   [HTTPS Everywhere](#https-everywhere)
    *   [Least Privilege](#least-privilege)
    *   [Data Encryption](#data-encryption)
8.  [Technology Stack](#8-technology-stack)
9.  [Deployment Ideatosystem](#9-deployment-ideatosystem)
    *   [Cloud Provider](#cloud-provider)
    *   [Infrastructure as Code (IaC)](#infrastructure-as-code-iac)
    *   [CI/CD Pipeline](#cicd-pipeline)
    *   [Deployment Strategy](#deployment-strategy)
    *   [Networking](#networking)
10. [Monitoring & Observability](#10-monitoring--observability)
    *   [Structured Logging](#structured-logging)
    *   [Metrics & APM](#metrics--apm)
    *   [Health Checks](#health-checks)
    *   [Alerting](#alerting)
    *   [Dashboarding](#dashboarding)
11. [Cost Estimation](#11-cost-estimation)
    *   [AWS Services Breakdown](#aws-services-breakdown)
    *   [Total Estimated Monthly Cost](#total-estimated-monthly-cost)
12. [Trade-offs & Limitations](#12-trade-offs--limitations)
    *   [Modular Monolith Architecture](#modular-monolith-architecture)
    *   [Managed Cloud Services](#managed-cloud-services)
    *   [Initial Observability Scope](#initial-observability-scope)
    *   [No Global Distribution](#no-global-distribution)
13. [Future Considerations](#13-future-considerations)
    *   [Microservices Evolution](#microservices-evolution-1)
    *   [Advanced AI/ML Pipelines](#advanced-aiml-pipelines)
    *   [Real-time Reporting & UI](#real-time-reporting--ui)
    *   [Enhanced Observability](#enhanced-observability)
    *   [Specialized Data Stores](#specialized-data-stores)
    *   [Global Distribution & Disaster Recovery](#global-distribution--disaster-recovery)

---

## 1. Executive Summary

This document details the production-ready system design for the "Automated Quant-Finance Codebase Documentation & Quality Workflow CLI." The project aims to provide a robust, platform-agnostic CLI for analyzing, documenting, and assessing the quality/security of quant-finance codebases, generating professional documentation, and quality reports.

The chosen architecture is a **modular monolith**, containerized, and deployed on a cloud platform (e.g., AWS). This approach offers a balance between rapid development, simplified deployment, and scalability for a startup environment, allowing it to serve 100s to 10K+ users with peak loads up to 800 RPS and bursts. The modular design facilitates future decomposition into microservices as needed, without incurring initial microservice overhead.

The design prioritizes:
*   **Scalability:** Horizontal scaling for stateless services, read replicas and caching for databases, and asynchronous processing for long-running tasks.
*   **Maintainability:** Clear component separation, containerization, Infrastructure as Code, and CI/CD pipelines.
*   **Security:** OAuth 2.0/OpenID Connect, rate limiting, input validation, secrets management, and encrypted communication.
*   **Cost-effectiveness:** Leveraging managed cloud services to minimize operational overhead while staying within a $500-2000/month budget.

This document explicitly outlines the core components, data models, API contracts, deployment strategies, and monitoring practices necessary for a robust production system, while also identifying key trade-offs and a clear upgrade path for future growth.

## 2. System Ideatosystem

The system leverages a **Modular Monolith** architecture for its backend services, deployed as distinct containerized components behind a load balancer. This allows for logical separation of concerns, easier development and deployment than a full microservices approach, and provides a clear path for future extraction of services if specific modules become bottlenecks. The CLI application remains client-side, interacting with the backend for advanced analysis and reporting features.

**Core Principles:**
*   **Containerization:** All backend services are containerized for consistent deployment across environments.
*   **Statelessness:** Backend API and worker services are stateless to facilitate horizontal scaling.
*   **Asynchronous Processing:** Heavy, long-running analysis tasks are offloaded to workers via a message queue to prevent API timeouts and ensure responsiveness.
*   **Managed Cloud Services:** Utilize cloud provider's managed services for databases, caching, queues, and object storage to reduce operational burden and ensure high availability.
*   **Security First:** Integrated authentication, authorization, and network security measures.

### High-Level Architecture Diagram

```mermaid
graph TD
    subgraph Client
        CLI_App[CLI Application (Local)]
    end

    subgraph AWS Cloud (e.g., us-east-1)
        direction LR
        User --- CName_Record(DNS (Route 53)) --> LoadBalancer(Application Load Balancer)
        LoadBalancer -->|HTTPS| API_Service_1(Backend API Service)
        LoadBalancer -->|HTTPS| API_Service_2(Backend API Service)
        LoadBalancer -->|HTTPS| API_Service_N(Backend API Service)
        
        API_Service_1 --> Auth_Provider(Auth Service / OAuth 2.0)
        API_Service_1 --> Redis(Managed Redis Cache)
        API_Service_1 --> SQS(Managed SQS Queue)
        API_Service_1 --> DB_Primary(Managed PostgreSQL Primary)
        
        SQS --- Worker_Service_1(Worker Service)
        SQS --- Worker_Service_2(Worker Service)
        SQS --- Worker_Service_N(Worker Service)
        
        Worker_Service_1 --> DB_Primary
        Worker_Service_1 --> S3(Managed S3 Object Storage)
        Worker_Service_1 --> LLM_API(External LLM API)
        
        DB_Primary ---(Replication)--> DB_Replica(Managed PostgreSQL Read Replica)
        
        API_Service_1 -- Metrics & Logs --> Monitoring(CloudWatch / Prometheus)
        Worker_Service_1 -- Metrics & Logs --> Monitoring
        
        subgraph Internal Network (VPC)
            API_Service_1
            API_Service_2
            API_Service_N
            Worker_Service_1
            Worker_Service_2
            Worker_Service_N
            Redis
            DB_Primary
            DB_Replica
        end
    end

    CLI_App --(External Network)--> CName_Record
    
    style User fill:#fff,stroke:#333,stroke-width:2px
    style CLI_App fill:#CCEEFF,stroke:#333,stroke-width:2px
    style CName_Record fill:#FFFACD,stroke:#333,stroke-width:1px
    style LoadBalancer fill:#ADD8E6,stroke:#333,stroke-width:2px
    style API_Service_1 fill:#B0E0E6,stroke:#333,stroke-width:1px
    style API_Service_2 fill:#B0E0E6,stroke:#333,stroke-width:1px
    style API_Service_N fill:#B0E0E6,stroke:#333,stroke-width:1px
    style Worker_Service_1 fill:#DAF7A6,stroke:#333,stroke-width:1px
    style Worker_Service_2 fill:#DAF7A6,stroke:#333,stroke-width:1px
    style Worker_Service_N fill:#DAF7A6,stroke:#333,stroke-width:1px
    style Auth_Provider fill:#FFD700,stroke:#333,stroke-width:1px
    style Redis fill:#FFB6C1,stroke:#333,stroke-width:1px
    style SQS fill:#FFC0CB,stroke:#333,stroke-width:1px
    style DB_Primary fill:#F08080,stroke:#333,stroke-width:1px
    style DB_Replica fill:#F08080,stroke:#333,stroke-width:1px
    style S3 fill:#A9A9A9,stroke:#333,stroke-width:1px
    style LLM_API fill:#D8BFD8,stroke:#333,stroke-width:1px
    style Monitoring fill:#E0FFFF,stroke:#333,stroke-width:1px
```

## 3. Component Design

This section details each component, optimized for production readiness, scalability, and security.

### CLI Application

*   **Description:** The primary user interface. Executes locally on the developer's machine. Handles local analysis (parsing AST, Git hooks), generates basic documentation/reports, and sends requests to the backend for advanced features (e.g., LLM-enhanced documentation, centralized quality reporting).
*   **Key Responsibilities:**
    *   Local codebase analysis (AST parsing, project structure recognition).
    *   Local documentation generation (Markdown, Mermaid/Graphviz diagrams).
    *   Static analysis execution (linting, type checking, security scans via local tools).
    *   Authentication with the backend (via API keys or OAuth flow).
    *   Sending analysis requests and configurations (`.kc-doc.yml`) to the backend API.
    *   Fetching and presenting results/reports from the backend.
    *   Git hook integration (pre-commit, pre-push).
*   **Technology Considerations:** Python (Click, Typer) for cross-platform compatibility and rich ecosystem for code analysis.

### Load Balancer / API Gateway

*   **Description:** The single entry point for all external traffic to the backend services. Distributes incoming requests across multiple instances of the Backend API Service.
*   **Key Responsibilities:**
    *   **Traffic Distribution:** Spreads requests evenly to prevent bottlenecks and improve throughput.
    *   **Health Checks:** Monitors the health of API service instances and routes traffic only to healthy ones, avoiding SPOF.
    *   **SSL/TLS Termination:** Handles encryption/decryption, offloading this burden from the backend services.
    *   **Basic Rate Limiting:** Protects against abuse and DDoS attacks.
    *   **Authentication (Initial pass):** Can perform initial JWT validation.
*   **Technology Recommendations:** AWS Application Load Balancer (ALB), Azure Application Gateway, Google Cloud Load Balancing. These are managed, highly available, and scalable.

### Backend API Service

*   **Description:** The core application logic, exposed via a RESTful API. Designed as a modular monolith, allowing for clear separation of business domains within a single deployable unit. This service is stateless.
*   **Key Modules (Logical, within the monolith):**
    *   **User & Organization Management:** Handles user profiles, roles, and potential multi-tenancy.
    *   **Repository Metadata:** Stores information about repositories registered for advanced analysis.
    *   **Analysis Orchestration:** Receives requests from the CLI, validates inputs, and dispatches long-running analysis tasks to the Message Queue.
    *   **Documentation & Report Retrieval:** Serves generated documentation and quality reports (often by providing links to S3).
    *   **LLM Proxy (Optional):** Routes requests to external LLM providers (e.g., OpenAI, Anthropic).
*   **Technology Recommendations:** Python (FastAPI/Django REST Framework), Go (Gin/Echo). Containerized (Docker). Deployed on AWS ECS Fargate, Azure Container Apps, or GCP Cloud Run for serverless container management.
*   **Scalability:** Horizontally scalable by adding more instances behind the Load Balancer.

### Worker Service

*   **Description:** Dedicated services for processing CPU-intensive and long-running tasks asynchronously. These workers consume tasks from the Message Queue.
*   **Key Responsibilities:**
    *   **Codebase Analysis:** Performs deeper static analysis (e.g., advanced security scans, dependency graph generation) that cannot be done locally or requires more resources.
    *   **LLM Interaction:** Executes prompts against LLMs for semantic analysis, enhanced documentation generation, and explanation of findings.
    *   **Diagram Generation:** Renders complex architecture/dataflow diagrams based on analysis results.
    *   **Report Aggregation:** Compiles and finalizes comprehensive quality and documentation reports.
    *   **Artifact Storage:** Uploads generated documentation and reports to Object Storage (S3).
*   **Technology Recommendations:** Python (Celery with Redis/SQS backend, or RQ), Go. Containerized (Docker). Deployed on AWS ECS Fargate, Azure Container Apps, or GCP Cloud Run.
*   **Scalability:** Horizontally scalable; multiple worker instances can process tasks in parallel.

### Authentication Service

*   **Description:** Manages user identities, authentication, and token issuance. For a startup, this can initially be integrated as a module within the Backend API Service. As the system grows, it can be externalized or replaced by a dedicated Identity Provider (IdP).
*   **Key Responsibilities:**
    *   User registration and login.
    *   OAuth 2.0 / OpenID Connect flow.
    *   Issuance and validation of JSON Web Tokens (JWTs).
    *   Password hashing and secure storage.
*   **Technology Recommendations:** Integrated into Backend (e.g., FastAPI with `python-jose` for JWTs) or a managed service like Auth0, AWS Cognito, Keycloak (self-hosted as a separate container).

### Database (PostgreSQL)

*   **Description:** The primary persistent data store for all structured application data.
*   **Key Responsibilities:**
    *   Storing user information, organization data.
    *   Repository metadata and configurations.
    *   Analysis job details, status, and parameters.
    *   Summaries of analysis results, quality findings.
    *   Links/pointers to generated documentation artifacts in Object Storage.
*   **Technology Recommendations:** PostgreSQL. Utilized as a **Managed Service** (e.g., AWS RDS for PostgreSQL, Azure Database for PostgreSQL, GCP Cloud SQL for PostgreSQL).
    *   **SPOF Avoidance:** Managed services provide automated backups, failover to a standby instance, and multi-AZ deployment options.
    *   **Scalability:** Supports **Read Replicas** for offloading read-heavy queries from the primary instance.

### Cache (Redis)

*   **Description:** An in-memory data store used for high-speed data retrieval, reducing the load on the primary database.
*   **Key Responsibilities:**
    *   Caching frequently accessed data (e.g., user profiles, recent report summaries, common configuration settings).
    *   Storing session tokens or temporary data (e.g., rate limit counters).
    *   Backend for task queues (if using Celery/RQ with Redis).
*   **Technology Recommendations:** Redis. Utilized as a **Managed Service** (e.g., AWS ElastiCache for Redis, Azure Cache for Redis, GCP Memorystore for Redis).
    *   **SPOF Avoidance:** Managed Redis offers replication and failover for high availability.

### Message Queue (SQS)

*   **Description:** A distributed message queue for asynchronous communication between the API Service (producer) and Worker Service (consumer).
*   **Key Responsibilities:**
    *   **Decoupling:** Separates the API's request handling from the long-running analysis tasks.
    *   **Reliability:** Ensures that tasks are eventually processed, even if workers fail temporarily.
    *   **Load Leveling:** Buffers incoming requests during traffic spikes, allowing workers to process them at their own pace.
    *   **Asynchronous Task Processing:** Facilitates background processing of analysis jobs.
*   **Technology Recommendations:** AWS SQS, Azure Service Bus, GCP Pub/Sub. These are fully managed, highly scalable, and highly available.
    *   **SPOF Avoidance:** Managed message queues are inherently distributed and highly available.

### Object Storage (S3)

*   **Description:** Highly durable, scalable, and cost-effective storage for unstructured data.
*   **Key Responsibilities:**
    *   Storing generated documentation files (README.md, ARCHITECTURE.md, DATAFLOW.md, MODULES.md).
    *   Archiving comprehensive quality reports and static analysis findings.
    *   Storing large diagrams or visual artifacts.
    *   Potentially storing raw codebase snapshots if remote analysis requires it (though for local-first, this is less common).
*   **Technology Recommendations:** AWS S3, Azure Blob Storage, GCP Cloud Storage.
    *   **SPOF Avoidance:** Managed object storage services are designed for extreme durability and availability across multiple availability zones.

## 4. Data Model

The data model is designed to support user and repository management, analysis job orchestration, and the storage of generated documentation and quality findings.

### Entities & Relationships

```mermaid
erDiagram
    USER {
        UUID id PK
        VARCHAR email UNIQUE
        VARCHAR password_hash
        TEXT api_key UNIQUE
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    ORGANIZATION {
        UUID id PK
        VARCHAR name UNIQUE
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    USER ||--o{ ORGANIZATION : "can belong to"

    REPOSITORY {
        UUID id PK
        UUID user_id FK "Owner"
        UUID organization_id FK "Optional owner"
        VARCHAR name
        VARCHAR vcs_url "e.g., git@github.com:..."
        TEXT configuration_yaml "Raw .kc-doc.yml content"
        TIMESTAMP last_analyzed_at
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    USER ||--o{ REPOSITORY : "owns"
    ORGANIZATION ||--o{ REPOSITORY : "owns"

    ANALYSIS_JOB {
        UUID id PK
        UUID repository_id FK
        UUID user_id FK "Who initiated"
        VARCHAR status ENUM("PENDING", "RUNNING", "COMPLETED", "FAILED")
        JSONB job_params "e.g., {'profile': 'quant-python', 'llm_mode': 'full'}"
        VARCHAR output_s3_prefix "S3 path for all job artifacts"
        TIMESTAMP initiated_at
        TIMESTAMP completed_at
        TEXT error_message
    }

    REPOSITORY ||--o{ ANALYSIS_JOB : "triggers"
    USER ||--o{ ANALYSIS_JOB : "initiates"

    QUALITY_FINDING {
        UUID id PK
        UUID analysis_job_id FK
        VARCHAR severity ENUM("CRITICAL", "HIGH", "MEDIUM", "LOW", "INFO")
        VARCHAR type ENUM("LINT", "SECURITY", "TYPE_ERROR", "METRIC")
        VARCHAR tool "e.g., bandit, pylint"
        VARCHAR file_path
        INTEGER line_number
        TEXT description
        JSONB details "Additional context (e.g., rule ID)"
        TIMESTAMP created_at
    }

    ANALYSIS_JOB ||--o{ QUALITY_FINDING : "generates"

    DOCUMENTATION_ARTIFACT {
        UUID id PK
        UUID analysis_job_id FK
        VARCHAR type ENUM("README", "ARCHITECTURE", "DATAFLOW", "MODULES", "QUALITY_REPORT_SUMMARY")
        TEXT s3_path "Full S3 path to artifact"
        TIMESTAMP created_at
    }

    ANALYSIS_JOB ||--o{ DOCUMENTATION_ARTIFACT : "generates"
```

### Indexing Strategy

*   `USER`: `email` (unique index), `id` (PK, implicit index)
*   `ORGANIZATION`: `name` (unique index), `id` (PK, implicit index)
*   `REPOSITORY`: `user_id`, `organization_id`, `last_analyzed_at`, `id` (PK, implicit index)
*   `ANALYSIS_JOB`: `repository_id`, `user_id`, `status`, `initiated_at`, `id` (PK, implicit index)
*   `QUALITY_FINDING`: `analysis_job_id`, `severity`, `type`, `id` (PK, implicit index)
*   `DOCUMENTATION_ARTIFACT`: `analysis_job_id`, `type`, `id` (PK, implicit index)

### Connection Pooling

*   **Implementation:** The Backend API and Worker services will use connection pooling (e.g., PgBouncer sidecar, or application-level pooling with SQLAlchemy in Python) to efficiently manage connections to the PostgreSQL database.
*   **Rationale:** Reduces connection overhead, improves performance, and prevents the database from being overwhelmed by too many simultaneous connections, especially during high traffic or burst scenarios.

### Caching Layer Strategy

*   **Purpose:** Reduce database load, improve API response times for frequently accessed data.
*   **Target Data:**
    *   **User & Organization Data:** Frequently requested user profiles, API keys, organization settings.
    *   **Recent Analysis Job Statuses:** For users checking the status of their ongoing jobs.
    *   **Summary Reports:** High-level summaries of quality reports or documentation overviews.
*   **Mechanism:**
    *   Utilize Redis with appropriate Time-To-Live (TTL) values.
    *   Cache-aside pattern: Application checks Redis first, if not found, fetches from DB, then populates Redis.
    *   Cache invalidation: Implement strategies for invalidating cache entries when underlying data changes (e.g., user updates profile, a new analysis job completes).

## 5. API Design

The API will be a RESTful JSON API, primarily consumed by the CLI application.

### Authentication

*   **Mechanism:** OAuth 2.0 with JSON Web Tokens (JWTs).
*   **Flow:**
    1.  CLI sends credentials (`email`/`password`) to `/auth/login`.
    2.  Backend authenticates user and returns an `access_token` (JWT) and a `refresh_token`.
    3.  CLI includes the `access_token` in the `Authorization: Bearer <token>` header for all subsequent API requests.
    4.  The API Gateway or Backend API validates the JWT signature and expiration.
    5.  When the `access_token` expires, CLI uses the `refresh_token` to obtain a new `access_token` from `/auth/refresh`.
*   **API Keys:** For automated workflows (e.g., CI/CD), dedicated API keys can be generated by users via a web portal or CLI. These API keys would be long-lived tokens with specific permissions.

### Key Endpoints & Contracts

**Base URL:** `https://api.kc-doc.com/v1` (example)

**1. Authentication & User Management**
*   `POST /auth/login`
    *   **Request:** `{"email": "...", "password": "..."}`
    *   **Response:** `{"access_token": "...", "refresh_token": "...", "expires_in": 3600}`
*   `POST /auth/refresh`
    *   **Request:** `{"refresh_token": "..."}`
    *   **Response:** `{"access_token": "...", "expires_in": 3600}`
*   `GET /users/me` (Authenticated)
    *   **Response:** `{"id": "...", "email": "...", "organizations": [...]}`

**2. Repository Management**
*   `GET /repositories` (Authenticated)
    *   **Response:** `[{"id": "...", "name": "...", "vcs_url": "...", "last_analyzed_at": "..."}]`
*   `POST /repositories` (Authenticated)
    *   **Request:** `{"name": "...", "vcs_url": "...", "configuration_yaml": "..."}`
    *   **Response:** `{"id": "...", "name": "..."}` (201 Created)
*   `GET /repositories/{repository_id}` (Authenticated)
    *   **Response:** `{"id": "...", "name": "...", "vcs_url": "...", "configuration_yaml": "...", "last_analyzed_at": "..."}`
*   `PUT /repositories/{repository_id}` (Authenticated)
    *   **Request:** `{"name": "...", "vcs_url": "...", "configuration_yaml": "..."}`
    *   **Response:** `{"id": "...", "name": "..."}`

**3. Analysis Job Orchestration**
*   `POST /repositories/{repository_id}/analyze` (Authenticated)
    *   **Request:** `{"profile": "quant-python", "llm_mode": "full", "force_reanalyze": false}`
    *   **Response:** `{"job_id": "...", "status": "PENDING", "message": "Analysis initiated."}` (202 Accepted)
*   `GET /analysis-jobs/{job_id}` (Authenticated)
    *   **Response:** `{"job_id": "...", "repository_id": "...", "status": "COMPLETED", "initiated_at": "...", "completed_at": "...", "error_message": null}`

**4. Documentation & Quality Reports**
*   `GET /analysis-jobs/{job_id}/artifacts` (Authenticated)
    *   **Response:** `[{"type": "README", "s3_path": "...", "created_at": "..."}, {"type": "QUALITY_REPORT_SUMMARY", "s3_path": "...", "created_at": "..."}]`
*   `GET /analysis-jobs/{job_id}/artifacts/{artifact_type}/download` (Authenticated)
    *   **Response:** Redirects to a pre-signed S3 URL for direct download of the artifact.
*   `GET /analysis-jobs/{job_id}/quality-findings` (Authenticated)
    *   **Response:** `[{"id": "...", "severity": "HIGH", "type": "SECURITY", "file_path": "...", "line_number": 123, "description": "...", "tool": "..."}, ...]`

**Error Handling:**
*   Standard HTTP status codes (e.g., 200 OK, 201 Created, 202 Accepted, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 500 Internal Server Error).
*   Error responses will be in JSON format: `{"code": "error_code", "message": "Human-readable error description"}`

## 6. Scalability Strategy

The system is designed for horizontal scaling across most components, leveraging cloud-native capabilities.

### Horizontal Scaling

*   **Stateless Services:** Both the Backend API Service and Worker Service are designed to be stateless. This means any instance can handle any request or task, making horizontal scaling straightforward.
*   **Container Orchestration:** Deploying services on platforms like AWS ECS Fargate, Azure Container Apps, or GCP Cloud Run allows for automatic scaling of service instances based on metrics such as CPU utilization, memory usage, or request queue depth.
*   **Load Balancing:** The Load Balancer automatically distributes incoming traffic across all healthy instances of the Backend API Service.

### Database Scaling

*   **Read Replicas:** For read-heavy workloads (e.g., fetching reports, listing repositories), the PostgreSQL database will utilize one or more read replicas. The Backend API Service can be configured to direct read queries to these replicas, significantly offloading the primary database.
*   **Connection Pooling:** Implemented within the application (and potentially via a sidecar like PgBouncer) to efficiently manage database connections, reducing overhead and preventing resource exhaustion on the database server.
*   **Indexing:** Proper indexing (as described in the Data Model) is crucial for optimizing query performance, ensuring the database remains responsive under load.

### Caching

*   **Reduced DB Load:** The Redis cache reduces the number of direct database queries for frequently accessed data, thereby lessening the load on the primary and replica databases.
*   **Faster Responses:** Caching common API responses or data chunks improves latency for end-users.

### Asynchronous Processing

*   **Message Queue:** Long-running and resource-intensive tasks (e.g., full codebase analysis, LLM calls) are offloaded to the Worker Service via the Message Queue. This ensures the API remains responsive to client requests and prevents timeouts.
*   **Decoupling:** The API and Worker services are decoupled, allowing them to scale independently. If analysis tasks spike, only the Worker Service needs to scale out, without affecting API responsiveness.

## 7. Security Considerations

Security is paramount, especially for a system handling proprietary codebase information and catering to financial institutions.

*   **Authentication & Authorization:**
    *   **OAuth 2.0 / OpenID Connect:** Used for user authentication, issuing JWTs for API access.
    *   **JWT Validation:** All backend services validate JWTs (signature, expiration, issuer) at the API Gateway level or within the service itself.
    *   **Role-Based Access Control (RBAC):** Implement basic roles (e.g., 'user', 'organization_admin') and enforce permissions based on these roles for resource access (e.g., `user_id` in repository table).
*   **Rate Limiting:**
    *   Implemented at the **Load Balancer / API Gateway** level to protect against API abuse, brute-force attacks, and DDoS attempts. Configured to limit requests per IP address or per authenticated user token.
*   **Input Validation:**
    *   Strict validation of all incoming API requests (payloads, query parameters, headers) to prevent common vulnerabilities like SQL injection, XSS, and command injection.
    *   Server-side validation is essential, even if client-side validation exists.
*   **Secrets Management:**
    *   Sensitive credentials (database passwords, API keys for external services like LLMs, internal service-to-service keys) are stored securely using dedicated **Cloud Secret Managers** (e.g., AWS Secrets Manager, Azure Key Vault, GCP Secret Manager).
    *   No hardcoded secrets in the codebase. Environment variables are used for non-sensitive configurations.
*   **Security Headers:**
    *   API responses include relevant security headers (e.g., `X-Content-Type-Options`, `X-Frame-Options`, `Content-Security-Policy` if a UI serves static content) to mitigate browser-based attacks.
*   **HTTPS Everywhere:**
    *   All external communication (CLI to Load Balancer) and internal communication (Load Balancer to API, API to DB/Redis, Workers to DB/S3) is enforced over **TLS/SSL**.
    *   Managed services provide TLS by default.
*   **Least Privilege:**
    *   All services and IAM roles/service accounts are configured with the minimum necessary permissions required to perform their functions.
    *   Network security groups/firewalls restrict access between components (e.g., only Backend API can access DB, only Workers can access SQS queue for consuming).
*   **Data Encryption:**
    *   **Encryption at Rest:** All data stored in the database, object storage, and cache is encrypted at rest using managed service features (e.g., AWS RDS encryption, S3 encryption, ElastiCache encryption).
    *   **Encryption in Transit:** Ensured by HTTPS/TLS for all communication paths.
*   **Regular Security Audits & Updates:**
    *   Regular scanning of container images for vulnerabilities.
    *   Keeping all dependencies, operating systems, and application frameworks up-to-date to patch known vulnerabilities.
    *   Periodic security reviews of the codebase and infrastructure.

## 8. Technology Stack

The recommended technology stack balances development speed, robustness, and cost-effectiveness for a production-ready startup environment.

| Component                 | Recommended Technology             | Justification for PRODUCTION                                                                                                                                                                   |
| :------------------------ | :--------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CLI Application**       | Python (Click/Typer)               | Rich ecosystem for parsing, static analysis tools; cross-platform compatibility; familiar for quant developers.                                                                                  |
| **Backend API Service**   | Python (FastAPI or Django REST)    | High productivity, strong async capabilities (FastAPI), mature ORM and ecosystem (Django). Excellent for data processing and integration with ML/LLM tools. Containerized.                       |
| **Worker Service**        | Python (Celery/RQ)                 | Seamless integration with Python backend; robust for asynchronous task processing; good error handling and retry mechanisms. Containerized.                                                    |
| **Database**              | PostgreSQL (Managed service e.g., AWS RDS) | Open-source, ACID compliant, robust, extensible, excellent JSONB support. Managed services provide high availability, backups, and read replicas.                                          |
| **Cache**                 | Redis (Managed service e.g., AWS ElastiCache) | High-performance in-memory data store for caching, session management, and rate limiting. Managed services offer high availability and ease of operation.                                 |
| **Message Queue**         | AWS SQS / GCP Pub/Sub / Azure Service Bus | Fully managed, highly scalable, reliable, and cost-effective messaging service. Eliminates the operational overhead of self-hosting a message broker.                                   |
| **Object Storage**        | AWS S3 / GCP Cloud Storage / Azure Blob Storage | Extremely durable, highly scalable, cost-effective for storing large artifacts (documentation, reports). Provides global availability.                                                       |
| **Load Balancer**         | AWS ALB / Azure App Gateway / GCP Load Balancer | Highly available, scalable, managed service. Provides SSL termination, health checks, and intelligent routing.                                                                         |
| **Containerization**      | Docker                             | Industry standard for packaging applications and their dependencies, ensuring consistent environments from development to production.                                                        |
| **Container Orchestration** | AWS ECS Fargate / GCP Cloud Run / Azure Container Apps | Serverless container platforms. Significantly reduces operational overhead for managing servers and Kubernetes clusters, ideal for startups.                                               |
| **Authentication**        | JWTs (OAuth2/OIDC standards) / AWS Cognito / Auth0 | Standard, secure token-based authentication. Managed services like Cognito/Auth0 offload complex auth requirements, providing features like MFA, SSO, and social logins.                  |
| **Infrastructure as Code**| Terraform / AWS CloudFormation     | Enables declarative, version-controlled, and repeatable infrastructure provisioning. Reduces manual errors and speeds up deployments.                                                    |
| **CI/CD**                 | GitHub Actions / GitLab CI / AWS CodePipeline | Automates the build, test, and deployment process. Ensures faster, more reliable releases and consistent quality checks.                                                               |
| **Monitoring & Logging**  | AWS CloudWatch / Prometheus + Grafana / Datadog (basic) | CloudWatch for metrics, logs, and basic alarms. Prometheus/Grafana for advanced custom metrics. Datadog (basic tier) for APM and centralized observability. Structured logging (JSON). |

## 9. Deployment Ideatosystem

The deployment strategy focuses on leveraging managed cloud services for efficiency, scalability, and high availability, specifically using AWS as an example.

### Cloud Provider

*   **AWS (Amazon Web Services):** Chosen for its comprehensive suite of managed services, strong ecosystem, and proven scalability for various workloads. (Alternatives: Azure, GCP).

### Infrastructure as Code (IaC)

*   **Terraform:** All infrastructure components (VPC, subnets, load balancers, ECS services, RDS, ElastiCache, SQS, S3 buckets, IAM roles, security groups) will be defined using Terraform.
    *   **Benefits:** Reproducible environments, version control for infrastructure, faster provisioning, reduced manual errors.

### CI/CD Pipeline

*   **GitHub Actions:** Used to automate the build, test, and deployment process.
    *   **Workflow:**
        1.  **Code Commit:** Developer pushes code to GitHub repository.
        2.  **Linting & Unit Tests:** GitHub Actions runs automated linting and unit tests.
        3.  **Build Docker Image:** If tests pass, Docker images for Backend API and Worker services are built.
        4.  **Image Push:** Docker images are pushed to AWS ECR (Elastic Container Registry).
        5.  **Integration Tests:** (Optional but recommended for production) Run integration tests against a staging environment.
        6.  **Terraform Apply:** Terraform applies infrastructure changes (if any) and updates the ECS services to use the new Docker image versions.
        7.  **Deployment:** ECS performs a rolling update, deploying new containers without downtime.

### Deployment Strategy

*   **Containerized Services:**
    *   **AWS ECS Fargate:** Backend API Service and Worker Service containers are deployed on Fargate. This is a serverless compute engine for containers, eliminating the need to provision, scale, and manage servers.
    *   **Health Checks:** ECS monitors container health, automatically replacing unhealthy instances.
    *   **Rolling Updates:** New versions are deployed without service interruption.
*   **Managed Database:**
    *   **AWS RDS for PostgreSQL:** Provisioned with multi-AZ deployment for high availability and automated failover.
    *   **Read Replicas:** Configured to offload read traffic.
*   **Managed Cache:**
    *   **AWS ElastiCache for Redis:** Configured with replication for high availability.
*   **Managed Message Queue:**
    *   **AWS SQS:** Standard queue used, automatically scales.
*   **Managed Object Storage:**
    *   **AWS S3:** Buckets for storing artifacts, with appropriate access policies.

### Networking

*   **Virtual Private Cloud (VPC):** A dedicated, isolated virtual network in AWS.
*   **Subnets:**
    *   **Public Subnets:** For the Application Load Balancer.
    *   **Private Subnets:** For ECS Fargate tasks (API and Workers), RDS instances, and ElastiCache. This ensures that databases and application servers are not directly accessible from the internet.
*   **Security Groups:** Act as virtual firewalls, strictly controlling inbound and outbound traffic for each component (e.g., only Load Balancer can talk to API, only API/Workers can talk to DB on port 5432).
*   **Route 53:** AWS's DNS service for managing domain names and routing traffic to the Load Balancer.

## 10. Monitoring & Observability

Comprehensive monitoring and observability are crucial for understanding system health, performance, and for rapid debugging in a production environment.

### Structured Logging

*   **Format:** All application logs (Backend API, Workers) will be emitted in **JSON format**.
*   **Content:** Logs will include essential metadata such as timestamp, service name, log level, request ID (for correlating requests across services), user ID (if applicable), and a clear message.
*   **Centralized Collection:** Logs are sent to **AWS CloudWatch Logs**. From there, they can be streamed to a centralized logging platform like Splunk, Datadog Logs, or an ELK (Elasticsearch, Logstash, Kibana) stack for advanced querying, alerting, and dashboarding.

### Metrics & APM

*   **Cloud Metrics:** **AWS CloudWatch** will collect basic infrastructure metrics (CPU, Memory, Network I/O for Fargate tasks, RDS, ElastiCache, SQS queue depth).
*   **Application Metrics:**
    *   Custom application metrics (e.g., request count, request latency per endpoint, error rates, task processing time, queue consumption rate) will be exposed via Prometheus-compatible endpoints or sent directly to CloudWatch custom metrics.
    *   **APM (Application Performance Monitoring) Basics:** Initially, integrate a basic APM tool (e.g., **Datadog APM Lite** or **New Relic One Free Tier**) to gain insights into service performance, trace requests across the API and workers, and identify performance bottlenecks. This might involve instrumenting the code with OpenTelemetry.
*   **Prometheus + Grafana (Optional but Recommended):** For more granular custom metrics and flexible dashboarding.

### Health Checks

*   **HTTP Endpoints:** Both the Backend API Service and Worker Service will expose `/health` and `/ready` HTTP endpoints.
    *   `/health`: A simple check to confirm the service is running.
    *   `/ready`: A more comprehensive check that verifies database connectivity, cache connectivity, and other critical dependencies.
*   **Load Balancer Integration:** The Load Balancer uses `/health` checks to determine which instances are healthy and can receive traffic, preventing SPOF.
*   **ECS Integration:** ECS uses these checks to determine if a new task is ready to receive traffic and if an existing task is still healthy.

### Alerting

*   **CloudWatch Alarms:** Set up alarms on critical CloudWatch metrics:
    *   High 5xx error rates on the Load Balancer.
    *   High CPU/memory utilization on API or Worker services.
    *   SQS queue message backlog (indicating slow processing).
    *   Database connection errors or high latency.
    *   Low disk space on the database.
*   **APM Alerts:** Configure alerts for application-specific issues (e.g., slow API endpoints, high error rates for specific business logic).
*   **Notification Channels:** Alerts will be configured to notify relevant teams via Slack, email, or PagerDuty for critical issues.

### Dashboarding

*   **CloudWatch Dashboards:** For a quick overview of critical infrastructure metrics.
*   **Grafana:** For combining metrics from various sources (CloudWatch, Prometheus, APM) into customizable dashboards, offering a unified view of system health and performance.

## 11. Cost Estimation

This estimation targets the $500-2000/month range using AWS services, optimized for startup scale (10k+ users, up to 800 peak RPS). Prices are approximate and based on typical `us-east-1` (N. Virginia) on-demand rates, assuming modest data transfer.

| Service Category    | AWS Service             | Configuration / Usage                                              | Estimated Monthly Cost (USD) | Notes                                                                                   |
| :------------------ | :---------------------- | :----------------------------------------------------------------- | :--------------------------- | :-------------------------------------------------------------------------------------- |
| **Compute**         | AWS ECS Fargate         | **Backend API:** 3-4 tasks, 0.5 vCPU, 1GB RAM each. 24/7.        | $150 - $250                  | Scales up/down based on load; average usage assumed.                                    |
|                     |                         | **Worker Service:** 3-5 tasks, 0.5 vCPU, 1GB RAM each. 24/7 (can be bursty). | $150 - $300                  | CPU-intensive tasks; can scale significantly during bursts.                             |
| **Database**        | AWS RDS for PostgreSQL  | `db.t3.medium` (2 vCPU, 4GB RAM) Primary, Multi-AZ.                | $120 - $180                  | Includes compute, I/O, backup storage. Multi-AZ adds ~50% cost for HA.                  |
|                     |                         | `db.t3.small` (2 vCPU, 2GB RAM) Read Replica.                      | $50 - $80                    | Offloads read queries, improves performance.                                            |
|                     |                         | **Storage (GP2 SSD):** 100GB (for primary + replica).             | $10 - $20                    | Sufficient for initial data.                                                            |
| **Cache**           | AWS ElastiCache (Redis) | `cache.t3.small` (1-2GB RAM), Multi-AZ (replication).              | $40 - $70                    | Improves read performance, reduces DB load.                                             |
| **Messaging**       | AWS SQS                 | 10 million requests/month (includes initial free tier).            | $1 - $5                      | Very cost-effective for high throughput.                                                |
| **Object Storage**  | AWS S3                  | 500GB Standard storage, 1TB data transfer out.                     | $15 - $30                    | For storing docs/reports. Data transfer is often the larger S3 cost.                    |
| **Networking**      | AWS ALB                 | 1 Load Balancer (approx. 200 hours/month + LCU usage).             | $25 - $40                    | Essential for traffic distribution and HA.                                              |
|                     |                         | Data Transfer Out (to internet, after free tier)                   | $10 - $30                    | Assumes egress traffic from ALB, S3.                                                    |
|                     | AWS Route 53            | DNS hosting for 1-2 domains, 10M queries.                          | $1 - $3                      | Reliable global DNS.                                                                    |
| **Monitoring/Logs** | AWS CloudWatch          | Logs ingestion (100GB), Metrics (custom, 100).                     | $20 - $50                    | Centralized logging and monitoring; scales with usage.                                  |
| **Security/Misc.**  | AWS Secrets Manager     | 10 secrets.                                                        | $5 - $10                     | Secure storage for critical credentials.                                                |
|                     | IAM (Identity)          | Free                                                               | $0                           | Core security.                                                                          |
| **Sub-Total (AWS)** |                         |                                                                    | **$600 - $1078**             |                                                                                         |
| **External (LLM)**  | OpenAI/Anthropic/etc.   | 5 million tokens/month (modest usage, variable per provider/model) | $50 - $200                   | Highly variable based on actual LLM usage and chosen model.                             |
| **APM (Basic)**     | Datadog/New Relic       | Starter/Free Tier                                                  | $0 - $50                     | Essential for observability, can grow significantly.                                    |
| **TOTAL ESTIMATE**  |                         |                                                                    | **$650 - $1328**             | Fits comfortably within the $500-$2000/month target. Allows significant growth headroom. |

*Note: Costs are estimates and can vary based on actual usage, specific AWS region, pricing updates, and negotiation. Significant LLM usage or advanced APM features could increase costs.*

## 12. Trade-offs & Limitations

Designing for production with a specific budget and scale always involves making strategic trade-offs.

*   **Modular Monolith Architecture:**
    *   **Pro:** Simpler to develop, deploy, and manage initially compared to a full microservices architecture. Reduces operational overhead for a small team.
    *   **Con:** Scaling an entire monolithic application just because one module is heavily utilized can be inefficient. Potential for tight coupling if module boundaries aren't strictly enforced. Can become a bottleneck if not eventually split.
    *   **Limitation:** While modular, scaling is often at the application level. If one specific analysis type or LLM interaction becomes orders of magnitude more popular, it might necessitate splitting that specific logic into its own microservice earlier than planned.
*   **Managed Cloud Services:**
    *   **Pro:** High availability, automated backups, patching, scaling, and reduced operational burden. "You pay for what you use."
    *   **Con:** Vendor lock-in. Less granular control over underlying infrastructure compared to self-hosting. Costs can become higher at extreme scales compared to highly optimized self-hosted solutions (but not for our target scale).
    *   **Limitation:** Reliance on AWS (or chosen cloud provider) for infrastructure health and new feature adoption.
*   **Initial Observability Scope (Basic APM):**
    *   **Pro:** Cost-effective start, provides essential metrics, logs, and basic tracing.
    *   **Con:** May lack deep distributed tracing, granular custom metrics, or advanced anomaly detection out-of-the-box compared to a fully instrumented and integrated enterprise APM solution.
    *   **Limitation:** Debugging complex, intermittent issues across multiple asynchronous components might be more challenging until further investment in advanced tracing and log correlation.
*   **No Global Distribution / Multi-Region Disaster Recovery:**
    *   **Pro:** Simpler infrastructure, lower cost, sufficient for initial production needs serving a broad user base from a single region.
    *   **Con:** The entire service would be unavailable if the primary cloud region experiences a widespread outage. Increased latency for geographically distant users.
    *   **Limitation:** Not designed for 99.99%+ uptime guarantees or sub-100ms latency for a global user base.
*   **CLI as Primary Interface:**
    *   **Pro:** Direct developer workflow integration, powerful for automation.
    *   **Con:** Limited accessibility for non-technical users or for broad administrative overview. Lacks a rich visual interface for exploring reports or configuring complex settings.
    *   **Limitation:** No dedicated web UI for centralized management or interactive reporting is part of the initial scope. Users might rely on downloaded Markdown/HTML for reports.

## 13. Future Considerations

This section outlines a clear upgrade path and potential enhancements as the system matures, user base grows, and business needs evolve.

*   **Microservices Evolution:**
    *   **Strategy:** Identify hot modules or independent business domains within the modular monolith that experience high load or require independent scaling/development cycles.
    *   **Candidates:**
        *   **LLM Integration Service:** Extract a dedicated microservice for all LLM interactions, allowing independent scaling, model swapping, and potentially A/B testing of different LLM providers/models.
        *   **Static Analysis Service:** Decouple the complex static analysis logic (parsing, linting, security scanning) into its own service, allowing for easier integration of new analysis tools or languages without impacting the core backend.
        *   **Reporting & Analytics Service:** Dedicated service for aggregating, processing, and serving historical analysis results and trends.
*   **Advanced AI/ML Pipelines:**
    *   Integrate more sophisticated ML models for vulnerability prediction, code smell detection, or optimizing documentation generation based on past feedback. This might involve dedicated ML training and inference pipelines.
    *   Utilize graph databases (e.g., Neo4j, Amazon Neptune) for advanced code dependency analysis and architectural insights.
*   **Real-time Reporting & UI:**
    *   Develop a **Web User Interface (UI)** for centralized repository management, interactive dashboards for quality reports, and visual exploration of architectural diagrams.
    *   Implement **WebSockets** for real-time updates on analysis job status, progress, and immediate feedback on security findings.
    *   Integrate a **CDN (Content Delivery Network)** like AWS CloudFront for serving static UI assets and cached documentation files globally, improving performance for web users.
*   **Enhanced Observability:**
    *   Full-fledged **Distributed Tracing:** Implement end-to-end request tracing (e.g., OpenTelemetry with Jaeger/Zipkin) to visualize and debug requests flowing across services, queues, and databases.
    *   **Advanced Alerting:** More sophisticated alerting rules based on behavioral patterns, predictive analytics, and integration with incident management tools (e.g., PagerDuty, Opsgenie).
    *   **Business Intelligence (BI) Tools:** Integrate with tools like AWS QuickSight or Tableau for deeper insights into usage patterns, popular analysis profiles, and overall platform health.
*   **Specialized Data Stores:**
    *   Consider dedicated **document databases** (e.g., MongoDB, AWS DynamoDB) for storing unstructured analysis results or highly flexible schemaless data.
    *   Explore **graph databases** for complex dependency mapping and architectural analysis where relational models are less efficient.
*   **Global Distribution & Disaster Recovery:**
    *   Implement a **multi-region deployment strategy** for enhanced disaster recovery, ensuring business continuity even if an entire cloud region fails.
    *   Utilize **global databases** (e.g., AWS Aurora Global Database) for highly available, low-latency access across regions.
    *   Implement **geo-DNS routing** (e.g., AWS Route 53 latency-based routing) to direct users to the nearest and lowest-latency region.
*   **CLI Plugin System & Ecosystem:**
    *   Open up the CLI with a robust plugin system, allowing third-party developers or internal teams to extend its capabilities with custom parsers, analysis tools, or documentation templates.
    *   Potentially create a marketplace for these extensions.
*   **Enterprise Features:**
    *   SAML/SSO (Single Sign-On) integration for corporate identity providers.
    *   Detailed audit trails and compliance reporting for regulatory requirements.
    *   Advanced access control with fine-grained permissions.

# Component Structure

## Nodes
- [custom] CLI End User: Interacts with the CLI application.
- [custom] CLI Application: Local codebase analysis, documentation generation.
- [custom] DNS Routing: Routes client requests to load balancer.
- [custom] Application Load Balancer: Distributes traffic to API services.
- [custom] Backend API Service: Core application logic, analysis orchestration.
- [custom] Authentication Service: Manages user identities and JWTs.
- [custom] Managed Redis Cache: High-speed data retrieval, reduces DB load.
- [custom] Managed SQS Queue: Decouples API from long-running tasks.
- [custom] PostgreSQL Primary DB: Main persistent data store for metadata.
- [custom] PostgreSQL Read Replica: Offloads read-heavy database queries.
- [custom] Worker Service: Processes intensive codebase analysis tasks.
- [custom] S3 Object Storage: Stores generated documents and reports.
- [custom] External LLM API: Third-party AI for enhanced analysis.
- [custom] Cloud Monitoring & Logs: Collects logs, metrics for observability.

## Data Flow
- CLI End User -> CLI Application (Uses)
- CLI Application -> DNS Routing (Makes API requests)
- DNS Routing -> Application Load Balancer (Routes traffic)
- Application Load Balancer -> Backend API Service (HTTPS API calls)
- Backend API Service -> Authentication Service (Authenticates user)
- Backend API Service -> Managed Redis Cache (Caches data)
- Backend API Service -> Managed SQS Queue (Dispatches job)
- Backend API Service -> PostgreSQL Primary DB (Writes/updates data)
- Backend API Service -> PostgreSQL Read Replica (Reads data)
- Managed SQS Queue -> Worker Service (Consumes task)
- Worker Service -> PostgreSQL Primary DB (Updates job status)
- Worker Service -> S3 Object Storage (Uploads artifacts)
- Worker Service -> External LLM API (Queries LLM)
- Backend API Service -> Cloud Monitoring & Logs (Emits logs/metrics)
- Worker Service -> Cloud Monitoring & Logs (Emits logs/metrics)
- PostgreSQL Primary DB -> PostgreSQL Read Replica (Data replication)
