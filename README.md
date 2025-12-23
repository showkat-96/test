# R&B Supply Chain Management System - Architecture

## System Overview Architecture

```mermaid
graph TB
    subgraph "External Systems"
        RMS[RMS - Retail Merchandise System]
        WMS[WMS - Warehouse Management System]
        TMS[TMS - Transport Management System]
    end

    subgraph "Frontend Layer"
        subgraph "React Application"
            Login[Login Component]
            Dashboard[Vendor Dashboard]
            POList[Purchase Orders List]
            PODetails[PO Details & Actions]
            DocMgr[Document Manager]
            Layout[App Layout & Navigation]
        end

        subgraph "Frontend Services"
            AuthCtx[Auth Context]
            APIService[API Service]
            Router[React Router]
        end
    end

    subgraph "Backend Layer"
        subgraph "API Gateway"
            Express[Express.js Server]
            AuthMW[Auth Middleware]
            CORS[CORS & Security]
            RateLimit[Rate Limiting]
        end

        subgraph "Controllers"
            AuthCtrl[Auth Controller]
            SupplierCtrl[Supplier Portal Controller]
            DocCtrl[Document Controller]
            MasterDataCtrl[Master Data Controller]
            NotificationCtrl[Notification Controller]
        end

        subgraph "Services"
            UserSvc[User Service]
            MasterDataSvc[Master Data Service]
            DocSvc[Document Service]
            NotificationSvc[Notification Service]
            PermissionSvc[Permission Service]
            AuditSvc[Audit Service]
        end

        subgraph "Middleware"
            JWT[JWT Validation]
            RBAC[Role-Based Access Control]
            Audit[Audit Logging]
            ErrorHandler[Error Handler]
        end
    end

    subgraph "Data Layer"
        subgraph "PostgreSQL Database"
            UserTables[(Users & Permissions)]
            POTables[(Purchase Orders & Items)]
            DocTables[(Documents & Uploads)]
            AuditTables[(Audit Logs)]
            NotificationTables[(Notifications)]
        end

        subgraph "File Storage"
            Uploads[Document Uploads]
            Barcodes[Generated Barcodes]
        end

        subgraph "Cache Layer"
            Redis[(Redis Cache)]
        end
    end

    %% Frontend to Backend Connections
    Login --> AuthCtrl
    Dashboard --> SupplierCtrl
    POList --> SupplierCtrl
    PODetails --> SupplierCtrl
    DocMgr --> DocCtrl

    AuthCtx --> APIService
    APIService --> Express

    %% Backend Internal Connections
    Express --> AuthMW
    AuthMW --> Controllers

    AuthCtrl --> UserSvc
    SupplierCtrl --> MasterDataSvc
    DocCtrl --> DocSvc
    MasterDataCtrl --> MasterDataSvc
    NotificationCtrl --> NotificationSvc

    %% Services to Data
    UserSvc --> UserTables
    MasterDataSvc --> POTables
    DocSvc --> DocTables
    DocSvc --> Uploads
    NotificationSvc --> NotificationTables
    AuditSvc --> AuditTables

    %% External System Integrations
    RMS --> MasterDataCtrl
    MasterDataSvc --> WMS
    MasterDataSvc --> TMS

    %% Cache Connections
    UserSvc --> Redis
    MasterDataSvc --> Redis

    %% Security Flow
    JWT --> RBAC
    RBAC --> Audit
    Audit --> AuditSvc

    style RMS fill:#ff9999
    style WMS fill:#ff9999
    style TMS fill:#ff9999
    style Express fill:#99ccff
    style PostgreSQL fill:#99ff99
    style Redis fill:#ffcc99
```

## Detailed Component Architecture

### Frontend Architecture

```mermaid
graph TB
    subgraph "React Frontend Application"
        subgraph "Pages"
            LoginPage[Login Page]
            DashboardPage[Vendor Dashboard]
            POPage[Purchase Orders Page]
        end

        subgraph "Components"
            LoginComp[Login Component]
            LayoutComp[App Layout]
            ProtectedRoute[Protected Route]
            DocManager[Document Manager]
            POTable[PO Table]
            PODrawer[PO Details Drawer]
        end

        subgraph "Context & State"
            AuthContext[Authentication Context]
            UserState[User State]
        end

        subgraph "Services"
            APIClient[API Client - Axios]
            AuthAPI[Auth API]
            SupplierAPI[Supplier Portal API]
        end

        subgraph "Routing"
            ReactRouter[React Router]
            RouteGuards[Route Guards]
        end
    end

    LoginPage --> LoginComp
    DashboardPage --> LayoutComp
    POPage --> LayoutComp

    LayoutComp --> POTable
    LayoutComp --> DocManager
    POTable --> PODrawer

    ProtectedRoute --> AuthContext
    AuthContext --> UserState

    LoginComp --> AuthAPI
    POTable --> SupplierAPI
    DocManager --> SupplierAPI

    AuthAPI --> APIClient
    SupplierAPI --> APIClient

    ReactRouter --> RouteGuards
    RouteGuards --> ProtectedRoute

    style AuthContext fill:#ffeb3b
    style APIClient fill:#2196f3
    style ReactRouter fill:#4caf50
```

### Backend Architecture

```mermaid
graph TB
    subgraph "Node.js Backend Server"
        subgraph "HTTP Layer"
            ExpressApp[Express Application]
            Middleware[Middleware Stack]
            Routes[Route Handlers]
        end

        subgraph "Controllers"
            AuthController[Authentication Controller]
            SupplierController[Supplier Portal Controller]
            DocumentController[Document Controller]
            MasterDataController[Master Data Controller]
        end

        subgraph "Services Layer"
            UserService[User Management Service]
            MasterDataService[Master Data Service]
            DocumentService[Document Service]
            NotificationService[Notification Service]
            PermissionService[Permission Service]
        end

        subgraph "Data Access Layer"
            DatabaseConfig[Database Configuration]
            QueryBuilder[Query Builder]
            ConnectionPool[Connection Pool]
        end

        subgraph "External Integrations"
            RMSIntegration[RMS Integration]
            WMSIntegration[WMS Integration]
            TMSIntegration[TMS Integration]
        end
    end

    ExpressApp --> Middleware
    Middleware --> Routes
    Routes --> Controllers

    AuthController --> UserService
    SupplierController --> MasterDataService
    DocumentController --> DocumentService
    MasterDataController --> MasterDataService

    UserService --> DatabaseConfig
    MasterDataService --> DatabaseConfig
    DocumentService --> DatabaseConfig
    NotificationService --> DatabaseConfig

    DatabaseConfig --> QueryBuilder
    QueryBuilder --> ConnectionPool

    MasterDataService --> RMSIntegration
    MasterDataService --> WMSIntegration
    MasterDataService --> TMSIntegration

    style ExpressApp fill:#68d391
    style Controllers fill:#63b3ed
    style Services fill:#f6ad55
    style DatabaseConfig fill:#fc8181
```

## Database Schema Architecture

```mermaid
erDiagram
    USERS {
        uuid id PK
        varchar email UK
        varchar password_hash
        varchar first_name
        varchar last_name
        user_role role
        varchar vendor_id FK
        boolean is_active
        timestamp last_login
        timestamp created_at
        timestamp updated_at
    }

    PURCHASE_ORDERS {
        uuid id PK
        varchar po_number UK
        varchar vendor_id FK
        varchar vendor_name
        varchar vendor_email
        decimal total_value
        varchar currency
        timestamp planned_eta
        timestamp planned_etd
        po_status status
        varchar rms_reference
        varchar season
        varchar category
        timestamp created_at
        timestamp updated_at
    }

    PO_ITEMS {
        uuid id PK
        uuid po_id FK
        varchar item_code
        text description
        integer quantity
        decimal unit_price
        decimal total_value
        varchar uom
        timestamp planned_delivery_date
        timestamp created_at
        timestamp updated_at
    }

    DOCUMENT_UPLOADS {
        uuid id PK
        uuid po_id FK
        varchar vendor_id FK
        varchar file_name
        varchar original_name
        bigint file_size
        varchar mime_type
        varchar upload_path
        document_type document_type
        boolean is_validated
        text[] validation_errors
        uuid uploaded_by FK
        timestamp created_at
        timestamp updated_at
    }

    PERMISSIONS {
        uuid id PK
        varchar name UK
        text description
        resource_type resource_type
        permission_action action
        timestamp created_at
    }

    ROLE_PERMISSIONS {
        uuid id PK
        user_role role
        uuid permission_id FK
        timestamp created_at
    }

    AUDIT_LOGS {
        uuid id PK
        uuid user_id FK
        varchar action
        resource_type resource_type
        varchar resource_id
        jsonb old_values
        jsonb new_values
        inet ip_address
        text user_agent
        timestamp created_at
    }

    VENDOR_NOTIFICATIONS {
        uuid id PK
        varchar vendor_id FK
        varchar vendor_email
        uuid po_id FK
        varchar notification_type
        varchar status
        timestamp sent_at
        text error_message
        integer retry_count
        timestamp created_at
        timestamp updated_at
    }

    USERS ||--o{ PURCHASE_ORDERS : "vendor_id"
    PURCHASE_ORDERS ||--o{ PO_ITEMS : "po_id"
    PURCHASE_ORDERS ||--o{ DOCUMENT_UPLOADS : "po_id"
    USERS ||--o{ DOCUMENT_UPLOADS : "uploaded_by"
    USERS ||--o{ AUDIT_LOGS : "user_id"
    PURCHASE_ORDERS ||--o{ VENDOR_NOTIFICATIONS : "po_id"
    PERMISSIONS ||--o{ ROLE_PERMISSIONS : "permission_id"
```

## Security Architecture

```mermaid
graph TB
    subgraph "Security Layers"
        subgraph "Network Security"
            HTTPS[HTTPS/TLS 1.3]
            CORS[CORS Policy]
            RateLimit[Rate Limiting]
        end

        subgraph "Authentication"
            JWT[JWT Tokens]
            Login[Login Validation]
            Session[Session Management]
        end

        subgraph "Authorization"
            RBAC[Role-Based Access Control]
            Permissions[Permission Matrix]
            ResourceAccess[Resource-Level Access]
        end

        subgraph "Data Protection"
            InputValidation[Input Validation]
            SQLInjection[SQL Injection Prevention]
            XSS[XSS Protection]
            FileValidation[File Upload Validation]
        end

        subgraph "Audit & Monitoring"
            AuditTrail[Audit Trail]
            ActionLogging[Action Logging]
            SecurityEvents[Security Event Monitoring]
        end
    end

    HTTPS --> CORS
    CORS --> RateLimit
    RateLimit --> JWT

    JWT --> Login
    Login --> Session
    Session --> RBAC

    RBAC --> Permissions
    Permissions --> ResourceAccess
    ResourceAccess --> InputValidation

    InputValidation --> SQLInjection
    SQLInjection --> XSS
    XSS --> FileValidation
    FileValidation --> AuditTrail

    AuditTrail --> ActionLogging
    ActionLogging --> SecurityEvents

    style HTTPS fill:#ff6b6b
    style JWT fill:#4ecdc4
    style RBAC fill:#45b7d1
    style AuditTrail fill:#96ceb4
```

## Data Flow Architecture

```mermaid
sequenceDiagram
    participant V as Vendor User
    participant F as Frontend
    participant A as API Gateway
    participant S as Services
    participant D as Database
    participant E as External Systems

    Note over V,E: Purchase Order Management Flow

    V->>F: Login Request
    F->>A: POST /auth/login
    A->>S: User Service
    S->>D: Validate Credentials
    D-->>S: User Data
    S-->>A: JWT Token
    A-->>F: Auth Response
    F-->>V: Dashboard

    V->>F: View Purchase Orders
    F->>A: GET /supplier-portal/purchase-orders
    A->>S: Master Data Service
    S->>D: Query POs by Vendor
    D-->>S: PO Data
    S-->>A: PO List
    A-->>F: PO Response
    F-->>V: PO Table

    V->>F: Accept Purchase Order
    F->>A: POST /supplier-portal/purchase-orders/:id/actions
    A->>S: Master Data Service
    S->>D: Update PO Status
    S->>D: Log Audit Trail
    D-->>S: Success
    S-->>A: Action Result
    A-->>F: Success Response
    F-->>V: Confirmation

    V->>F: Upload Document
    F->>A: POST /supplier-portal/purchase-orders/:id/documents
    A->>S: Document Service
    S->>D: Save Document Metadata
    S->>D: Store File
    D-->>S: Success
    S-->>A: Upload Result
    A-->>F: Upload Response
    F-->>V: Upload Confirmation

    Note over V,E: External System Integration

    E->>A: RMS PO Sync
    A->>S: Master Data Service
    S->>D: Create/Update PO
    S->>S: Notification Service
    S-->>E: Sync Confirmation
```

## Deployment Architecture

```mermaid
graph TB
    subgraph "Production Environment"
        subgraph "Load Balancer"
            LB[Application Load Balancer]
        end

        subgraph "Frontend Tier"
            CDN[CDN - Static Assets]
            WebServer[Web Server - Nginx]
        end

        subgraph "Application Tier"
            App1[Node.js App Instance 1]
            App2[Node.js App Instance 2]
            App3[Node.js App Instance 3]
        end

        subgraph "Database Tier"
            PrimaryDB[(PostgreSQL Primary)]
            ReplicaDB[(PostgreSQL Replica)]
            RedisCluster[(Redis Cluster)]
        end

        subgraph "File Storage"
            FileSystem[File System Storage]
            Backup[Backup Storage]
        end

        subgraph "Monitoring"
            Logs[Log Aggregation]
            Metrics[Metrics Collection]
            Alerts[Alert Manager]
        end
    end

    subgraph "External Services"
        RMSExt[RMS System]
        WMSExt[WMS System]
        TMSExt[TMS System]
        EmailSvc[Email Service]
    end

    LB --> WebServer
    WebServer --> CDN
    LB --> App1
    LB --> App2
    LB --> App3

    App1 --> PrimaryDB
    App2 --> PrimaryDB
    App3 --> PrimaryDB

    PrimaryDB --> ReplicaDB
    App1 --> RedisCluster
    App2 --> RedisCluster
    App3 --> RedisCluster

    App1 --> FileSystem
    App2 --> FileSystem
    App3 --> FileSystem
    FileSystem --> Backup

    App1 --> RMSExt
    App2 --> WMSExt
    App3 --> TMSExt
    App1 --> EmailSvc

    App1 --> Logs
    App2 --> Logs
    App3 --> Logs
    Logs --> Metrics
    Metrics --> Alerts

    style LB fill:#ff9999
    style PrimaryDB fill:#99ff99
    style RedisCluster fill:#ffcc99
    style CDN fill:#99ccff
```

## Key Architectural Principles

### 1. **Layered Architecture**

- **Presentation Layer**: React frontend with component-based architecture
- **API Layer**: Express.js with RESTful endpoints
- **Business Logic Layer**: Service classes with domain logic
- **Data Access Layer**: PostgreSQL with connection pooling

### 2. **Security-First Design**

- JWT-based authentication
- Role-based access control (RBAC)
- Input validation and sanitization
- Comprehensive audit logging
- File upload security

### 3. **Scalability**

- Stateless application design
- Database connection pooling
- Redis caching layer
- Horizontal scaling capability
- Load balancer ready

### 4. **Integration Architecture**

- RESTful API design
- External system webhooks
- Asynchronous processing
- Error handling and retry mechanisms
- Data synchronization patterns

### 5. **Data Management**

- Normalized database design
- Audit trail for all operations
- Document management system
- Backup and recovery strategies
- Data validation and integrity

This architecture provides a robust, scalable, and secure foundation for the R&B Supply Chain Management System, supporting the complete vendor portal workflow from authentication to document management.
