# 🧱 Multi-Database System Architecture and ER Diagram

This document describes the database architecture and table relationships for a distributed property and lease management system consisting of multiple services.

---

## 🏗️ **High-Level System Architecture Diagram**

```mermaid
graph TD
    subgraph User_Identity_Service["User / Identity Service (.NET)"]
        A1[users_db.users]
    end

    subgraph Property_Service["Property Registration Service (Spring Boot)"]
        B1[properties_db.properties]
    end

    subgraph Lease_Service["Lease & Tenancy Service (Spring Boot)"]
        C1[leases_db.leases]
    end

    subgraph Audit_Service["Audit Logging Service (ASP.NET Core)"]
        D1[audit_db.audit_logs]
    end

    subgraph Notification_Service["Notification Service (Spring Boot)"]
        E1[notifications_db.sent_notifications]
    end

    %% Relationships
    A1 -->|user_id| D1
    A1 -->|tenant_user_id| C1
    B1 -->|property_id| C1
    A1 -->|recipient_user_id| E1
    C1 -->|activity triggers| D1
    C1 -->|lease events| E1

    %% External Auth
    ext1[(Asgardio Identity Provider)]
    ext1 --> A1

    %% Interactions
    A1 -->|authenticates| Lease_Service
    B1 -->|manages property data| Lease_Service
    Lease_Service -->|creates audit events| Audit_Service
    Lease_Service -->|triggers notifications| Notification_Service
    Audit_Service -->|records system & user events| D1
```

---

## 🧱 **Entity Relationship (ER) Diagram**

```mermaid
erDiagram

    USERS {
        BIGINT id PK
        VARCHAR asgardio_user_id "Unique external user ID"
        VARCHAR email
        VARCHAR username
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    PROPERTIES {
        BIGINT id PK
        VARCHAR name
        TEXT address
        VARCHAR type
        VARCHAR status
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    LEASES {
        BIGINT id PK
        BIGINT property_id FK
        BIGINT tenant_user_id FK
        DATE start_date
        DATE end_date
        DECIMAL rent_amount
        VARCHAR status
        TIMESTAMP created_at
        TIMESTAMP updated_at
    }

    AUDIT_LOGS {
        BIGINT id PK
        TIMESTAMP timestamp
        VARCHAR service_name
        BIGINT user_id FK
        VARCHAR action
        JSON details
    }

    SENT_NOTIFICATIONS {
        BIGINT id PK
        BIGINT recipient_user_id FK
        VARCHAR type
        TEXT message
        VARCHAR status
        TIMESTAMP sent_at
    }

    %% Relationships
    USERS ||--o{ LEASES : "tenant_user_id"
    USERS ||--o{ AUDIT_LOGS : "user_id"
    USERS ||--o{ SENT_NOTIFICATIONS : "recipient_user_id"
    PROPERTIES ||--o{ LEASES : "property_id"
```

---

## 💡 **Summary**

| Service | Database | Key Table | Description |
|----------|-----------|------------|--------------|
| **User/Identity Service (.NET)** | `users_db` | `users` | Manages all authenticated users (linked via Asgardio ID). |
| **Property Service (Spring Boot)** | `properties_db` | `properties` | Manages property details (buildings, apartments, lands). |
| **Lease Service (Spring Boot)** | `leases_db` | `leases` | Links users to properties and tracks lease durations & rent. |
| **Audit Service (ASP.NET Core)** | `audit_db` | `audit_logs` | Tracks actions across all services for compliance. |
| **Notification Service (Spring Boot)** | `notifications_db` | `sent_notifications` | Stores sent/failed user notifications. |
